<template>
  <div ref="container" class="bedroom-model">
    <canvas ref="canvas" class="bedroom-canvas"></canvas>
    <div v-if="loading" class="loading">加载模型中…</div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as THREE from 'three'
import { GLTFLoader } from 'three/addons/loaders/GLTFLoader.js'
import { MeshoptDecoder } from 'three/addons/libs/meshopt_decoder.module.js'
import { RoomEnvironment } from 'three/addons/environments/RoomEnvironment.js'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

const props = defineProps({
  isDark: { type: Boolean, default: true }
})

const container = ref(null)
const canvas = ref(null)
const loading = ref(true)

let scene, camera, renderer, controls, model, floor, ambientLight
let rgbLights = []
let raycaster, pointer, doorHinge, screenMesh, trashGroup
let doorMeshes = []
let trashMeshes = []
let doorIsOpen = false, doorAnimating = false
let cameraAnimating = false
let focusedObject = null // 'screen' | 'trash' | null
let savedCameraPos = null, savedTarget = null
let downX = 0, downY = 0
const soundCache = {}

const DOOR_OPEN_ANGLE = -Math.PI / 2 // 门打开角度(负号反转方向)
const DOOR_ANIM_DURATION = 1000 // 开合动画时长(毫秒)
const CAMERA_MOVE_DURATION = 1500 // 相机缓动时长(毫秒)

function init() {
  const el = container.value
  const w = el.clientWidth || window.innerWidth
  const h = el.clientHeight || window.innerHeight

  scene = new THREE.Scene()

  camera = new THREE.PerspectiveCamera(45, w / h, 0.1, 500)

  renderer = new THREE.WebGLRenderer({ canvas: canvas.value, antialias: true, alpha: true })
  renderer.setSize(w, h)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  renderer.setClearColor(0x000000, 0)
  renderer.outputColorSpace = THREE.SRGBColorSpace
  renderer.toneMapping = THREE.ACESFilmicToneMapping
  renderer.toneMappingExposure = 1.0

  // 轨道控制:只保留鼠标左键拖拽旋转
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableRotate = true
  controls.enableZoom = false // 禁用滚轮缩放,让滚轮继续触发翻页
  controls.enablePan = false // 禁用平移
  controls.touches.ONE = THREE.TOUCH.ROTATE // 单指触摸旋转(手机可左右拖拽查看)
  controls.addEventListener('change', render) // 拖拽时按需重绘,不跑常驻循环

  // 光线投射(用于检测鼠标是否悬停/点击到门和屏幕)
  raycaster = new THREE.Raycaster()
  pointer = new THREE.Vector2()

  // 环境贴图:只给金属/粗糙度材质提供反射,不直接当主光用
  const pmrem = new THREE.PMREMGenerator(renderer)
  scene.environment = pmrem.fromScene(new RoomEnvironment(), 0.04).texture
  pmrem.dispose()

  // 环境光(颜色/强度随主题切换)
  ambientLight = new THREE.AmbientLight(0xffffff, 0.15)
  scene.add(ambientLight)

  // 加载模型(webp 纹理 + meshopt 几何压缩)
  const loader = new GLTFLoader()
  loader.setMeshoptDecoder(MeshoptDecoder)
  loader.load(
    '/models/room.glb',
    (gltf) => {
      model = gltf.scene
      scene.add(model)

      frameModel(model)
      addFloor(model)
      createRgbLights(model)
      setLighting(props.isDark)
      setupDoor(model)
      setupScreen(model)
      setupTrash(model)

      loading.value = false
      render()
    },
    undefined,
    (err) => {
      console.error('[BedroomModel] 模型加载失败:', err)
      loading.value = false
    }
  )
}

// 根据模型包围盒自动框住整个房间,并让旋转围绕模型中心
function frameModel(model) {
  const box = new THREE.Box3().setFromObject(model)
  const size = box.getSize(new THREE.Vector3())
  const center = box.getCenter(new THREE.Vector3())

  controls.target.copy(center)

  // 相机方向(3/4 视角),归一化后 dist 就是真实距离
  const dir = new THREE.Vector3(0.6, 0.25, 0.65).normalize()

  // 按模型宽高 + 视场角 + 屏幕宽高比计算距离,保证手机竖屏也完整展示(不裁剪)
  const fov = (camera.fov * Math.PI) / 180
  const tanHalf = Math.tan(fov / 2)
  const distX = (size.x / 2) / (tanHalf * camera.aspect)
  const distY = (size.y / 2) / tanHalf
  const dist = Math.max(distX, distY) * 1.6

  camera.position.copy(center).addScaledVector(dir, dist)
  camera.near = Math.max(dist * 0.05, 0.1)
  camera.far = dist * 20
  camera.updateProjectionMatrix()
  controls.update()
}

// 生成径向渐变贴图:中心为可见色,向外渐隐到透明
// 暗黑模式用青色霓虹,亮白模式用浅色阴影
function makeFloorTexture(dark) {
  const s = 512
  const canvas = document.createElement('canvas')
  canvas.width = canvas.height = s
  const ctx = canvas.getContext('2d')
  const g = ctx.createRadialGradient(s / 2, s / 2, 0, s / 2, s / 2, s / 2)
  const c = dark ? '0,229,255' : '0,0,0'
  const a = dark ? 0.7 : 0.55
  g.addColorStop(0, `rgba(${c},${a})`)
  g.addColorStop(0.6, `rgba(${c},${a * 0.45})`)
  g.addColorStop(1, `rgba(${c},0)`)
  ctx.fillStyle = g
  ctx.fillRect(0, 0, s, s)
  const tex = new THREE.CanvasTexture(canvas)
  tex.colorSpace = THREE.SRGBColorSpace
  return tex
}

// 在模型底部铺一层渐变地板
function addFloor(model) {
  const box = new THREE.Box3().setFromObject(model)
  const size = box.getSize(new THREE.Vector3())
  const center = box.getCenter(new THREE.Vector3())

  const side = Math.max(size.x, size.z) * 1.6
  const geo = new THREE.PlaneGeometry(side, side)
  const mat = new THREE.MeshBasicMaterial({
    map: makeFloorTexture(props.isDark),
    transparent: true,
    depthWrite: false
  })
  floor = new THREE.Mesh(geo, mat)
  floor.rotation.x = -Math.PI / 2 // 放平
  floor.position.set(center.x, box.min.y - 0.05, center.z)
  scene.add(floor)
}

// 在房间周围加两盏 RGB 霓虹灯(青色 + 品红),只在暗黑模式点亮
function createRgbLights(model) {
  const box = new THREE.Box3().setFromObject(model)
  const size = box.getSize(new THREE.Vector3())
  const center = box.getCenter(new THREE.Vector3())

  const cyan = new THREE.PointLight(0x00e5ff, 0, 30, 2)
  cyan.position.set(center.x - size.x * 0.6, center.y + size.y * 0.2, center.z + size.z * 0.6)
  scene.add(cyan)

  const magenta = new THREE.PointLight(0xff00e5, 0, 30, 2)
  magenta.position.set(center.x + size.x * 0.6, center.y + size.y * 0.2, center.z - size.z * 0.6)
  scene.add(magenta)

  rgbLights = [cyan, magenta]
}

// 根据主题统一调整:GLB 灯光、环境光、RGB 霓虹灯、地板颜色
function setLighting(dark) {
  // GLB 里的原始灯光(日光/点光):暗黑模式压低,让冷色霓虹成为主氛围
  if (model) {
    model.traverse((o) => {
      if (o.isDirectionalLight) {
        o.intensity = dark ? 1.5 : 3.0
      } else if (o.isPointLight) {
        o.intensity = dark ? 20 : 60
      }
    })
  }

  // 环境光:暗黑 = 暗紫蓝,亮白 = 白色
  if (ambientLight) {
    ambientLight.color.set(dark ? 0x1a1440 : 0xffffff)
    ambientLight.intensity = dark ? 0.2 : 0.15
  }

  // RGB 霓虹灯:暗黑点亮,亮白熄灭
  rgbLights.forEach((l) => {
    l.intensity = dark ? 70 : 0
  })

  // 地板颜色
  if (floor) {
    floor.material.map?.dispose()
    floor.material.map = makeFloorTexture(dark)
    floor.material.needsUpdate = true
  }
}

// ============ 交互对象查找 ============

// 找到门的铰链和所有网格
function setupDoor(model) {
  doorHinge = model.getObjectByName('Door_Hinge')
  doorMeshes = []
  if (doorHinge) {
    doorHinge.traverse((o) => {
      if (o.isMesh) doorMeshes.push(o)
    })
  }
}

// 找到屏幕网格(按 screen 材质名)
function findScreenMaterial() {
  let mat = null
  model.traverse((o) => {
    if (mat || !o.isMesh) return
    const mats = Array.isArray(o.material) ? o.material : [o.material]
    for (const m of mats) {
      if (m.name === 'screen') { mat = m; break }
    }
  })
  return mat
}

function setupScreen(model) {
  screenMesh = null
  model.traverse((o) => {
    if (screenMesh || !o.isMesh) return
    const mats = Array.isArray(o.material) ? o.material : [o.material]
    for (const m of mats) {
      if (m.name === 'screen') { screenMesh = o; break }
    }
  })
}

// 找到垃圾桶(Papelera_3)及其所有网格
function setupTrash(model) {
  trashGroup = model.getObjectByName('Papelera_3')
  trashMeshes = []
  if (trashGroup) {
    trashGroup.traverse((o) => {
      if (o.isMesh) trashMeshes.push(o)
    })
  }
}

// ============ 光线投射 ============

// 把鼠标屏幕坐标转成 NDC
function updatePointer(e) {
  const rect = canvas.value.getBoundingClientRect()
  pointer.x = ((e.clientX - rect.left) / rect.width) * 2 - 1
  pointer.y = -((e.clientY - rect.top) / rect.height) * 2 + 1
}

// 判断鼠标是否命中指定网格
function raycastMeshes(e, meshes) {
  if (!meshes.length || !camera) return false
  updatePointer(e)
  raycaster.setFromCamera(pointer, camera)
  return raycaster.intersectObjects(meshes, false).length > 0
}

function getInteractiveMeshes() {
  const list = [...doorMeshes, ...trashMeshes]
  if (screenMesh) list.push(screenMesh)
  return list
}

function onPointerMove(e) {
  const meshes = getInteractiveMeshes()
  if (!meshes.length) return
  canvas.value.style.cursor = raycastMeshes(e, meshes) ? 'pointer' : 'default'
}

function onPointerDown(e) {
  downX = e.clientX
  downY = e.clientY
}

function onClick(e) {
  // 拖动旋转时移动超过 5px 就不算点击,避免误触发
  const dist = Math.hypot(e.clientX - downX, e.clientY - downY)
  if (dist > 5) return
  if (doorAnimating || cameraAnimating) return

  if (raycastMeshes(e, doorMeshes)) {
    toggleDoor()
    return
  }
  if (screenMesh && raycastMeshes(e, [screenMesh])) {
    playSound('/breezy/a-loud-button-press-sound.mp3')
    focusOnScreen()
    return
  }
  if (trashMeshes.length && raycastMeshes(e, trashMeshes)) {
    playSound('/breezy/noisy-page-flip-sound.mp3')
    focusOnTrash()
  }
}

// ============ 门开合 ============

function toggleDoor() {
  playSound('/breezy/door-opening-sound.mp3')
  animateDoor(!doorIsOpen)
}

// 播放音效(自动缓存 Audio 元素,每次从头播放)
function playSound(url) {
  if (!soundCache[url]) {
    soundCache[url] = new Audio(url)
  }
  soundCache[url].currentTime = 0
  soundCache[url].play().catch(() => {})
}

// 开门/关门动画(绕铰链 Y 轴旋转,带缓动)
function animateDoor(open) {
  doorAnimating = true
  const start = doorHinge.rotation.y
  const target = open ? DOOR_OPEN_ANGLE : 0
  const t0 = performance.now()

  function step(now) {
    const t = Math.min((now - t0) / DOOR_ANIM_DURATION, 1)
    const eased = t < 0.5 ? 2 * t * t : 1 - Math.pow(-2 * t + 2, 2) / 2 // ease-in-out
    doorHinge.rotation.y = start + (target - start) * eased
    render()
    if (t < 1) {
      requestAnimationFrame(step)
    } else {
      doorIsOpen = open
      doorAnimating = false
    }
  }
  requestAnimationFrame(step)
}

// ============ 屏幕聚焦(相机移到屏幕前) ============

// 计算屏幕的世界法线(由首三角形叉乘得到)
function getScreenNormal() {
  const geom = screenMesh.geometry
  const pos = geom.attributes.position
  if (!pos || pos.count < 3) return new THREE.Vector3(0, 0, 1)
  const a = new THREE.Vector3().fromBufferAttribute(pos, 0)
  const b = new THREE.Vector3().fromBufferAttribute(pos, 1)
  const c = new THREE.Vector3().fromBufferAttribute(pos, 2)
  const n = new THREE.Vector3().subVectors(b, a).cross(new THREE.Vector3().subVectors(c, a)).normalize()
  // 让法线朝向当前相机(若相机在屏幕另一侧则翻转)
  const center = new THREE.Vector3()
  screenMesh.getWorldPosition(center)
  const toCam = camera.position.clone().sub(center)
  if (n.dot(toCam) < 0) n.negate()
  return n.transformDirection(screenMesh.matrixWorld)
}

// 相机聚焦到指定位置,再点同一物体退回原位
function focusOn(targetPos, targetLookAt, name) {
  if (focusedObject === name) {
    focusedObject = null
    if (savedCameraPos && savedTarget) {
      animateCamera(savedCameraPos, savedTarget, null)
    }
    return
  }
  savedCameraPos = camera.position.clone()
  savedTarget = controls.target.clone()
  focusedObject = name
  animateCamera(targetPos, targetLookAt, null)
}

// 点击屏幕:相机缓慢移动到屏幕正前方(图片直接显示在 3D 屏幕上)
function focusOnScreen() {
  const center = new THREE.Vector3()
  screenMesh.getWorldPosition(center)
  const normal = getScreenNormal()

  const box = new THREE.Box3().setFromObject(screenMesh)
  const size = box.getSize(new THREE.Vector3())
  // 按屏幕宽高 + 相机 FOV 精确计算距离,屏幕放大到占画面约 90%(留 10% 边距)
  const fov = (camera.fov * Math.PI) / 180
  const tanHalf = Math.tan(fov / 2)
  const distY = (size.y / 2) / tanHalf
  const distX = (size.x / 2) / (tanHalf * camera.aspect)
  const distance = Math.max(distX, distY, 0.3) * 1.2

  const targetPos = center.clone().add(normal.clone().multiplyScalar(distance))
  focusOn(targetPos, center, 'screen')
}

// 点击垃圾桶:相机移到桶口上方,俯视桶内
function focusOnTrash() {
  const box = new THREE.Box3().setFromObject(trashGroup)
  const size = box.getSize(new THREE.Vector3())
  const center = box.getCenter(new THREE.Vector3())
  const topY = box.max.y

  const maxDim = Math.max(size.x, size.z)
  const dist = Math.max(maxDim * 1.5, 0.3)
  const off = maxDim * 0.2 // 略偏一点,避免正上方云台锁

  const topCenter = new THREE.Vector3(center.x, topY, center.z)
  const targetPos = new THREE.Vector3(topCenter.x + off, topY + dist, topCenter.z + off)

  focusOn(targetPos, topCenter, 'trash')
}

// 相机缓动(从当前位置移动到目标位置,并看向目标点)
function animateCamera(targetPos, targetLookAt, onDone) {
  cameraAnimating = true
  controls.enabled = false // 动画期间禁用旋转

  const startPos = camera.position.clone()
  const startTarget = controls.target.clone()
  const t0 = performance.now()

  function step(now) {
    const t = Math.min((now - t0) / CAMERA_MOVE_DURATION, 1)
    const eased = t < 0.5 ? 2 * t * t : 1 - Math.pow(-2 * t + 2, 2) / 2 // ease-in-out
    camera.position.lerpVectors(startPos, targetPos, eased)
    controls.target.lerpVectors(startTarget, targetLookAt, eased)
    controls.update()
    render()
    if (t < 1) {
      requestAnimationFrame(step)
    } else {
      cameraAnimating = false
      controls.enabled = true
      if (onDone) onDone()
    }
  }
  requestAnimationFrame(step)
}

// ============ 换电脑屏幕贴图 ============

// 前端直接替换屏幕贴图,无需回 Blender 重新导出
function setScreenTexture(url) {
  const screenMat = findScreenMaterial()
  if (!screenMat) {
    console.warn('[BedroomModel] 未找到 screen 材质')
    return
  }

  const old = screenMat.map
  new THREE.TextureLoader().load(url, (tex) => {
    tex.colorSpace = THREE.SRGBColorSpace
    // 保留原贴图的 UV 变换(缩放/偏移/翻转),让新图正确贴合屏幕
    if (old) {
      tex.repeat.copy(old.repeat)
      tex.offset.copy(old.offset)
      tex.rotation = old.rotation
      tex.center.copy(old.center)
      tex.flipY = old.flipY
    }
    screenMat.map = tex
    screenMat.needsUpdate = true
    render()
  }, undefined, (err) => {
    console.error('[BedroomModel] 屏幕贴图加载失败:', err)
  })
}

// 主题切换时整体刷新氛围
watch(() => props.isDark, (dark) => {
  setLighting(dark)
  render()
})

function render() {
  renderer.render(scene, camera)
}

function handleResize() {
  const el = container.value
  const w = el.clientWidth || window.innerWidth
  const h = el.clientHeight || window.innerHeight
  camera.aspect = w / h
  camera.updateProjectionMatrix()
  renderer.setSize(w, h)
  render()
}

onMounted(() => {
  init()
  window.addEventListener('resize', handleResize)
  // 门/屏幕交互事件(挂在 canvas 上)
  canvas.value.addEventListener('pointermove', onPointerMove)
  canvas.value.addEventListener('pointerdown', onPointerDown)
  canvas.value.addEventListener('click', onClick)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  canvas.value.removeEventListener('pointermove', onPointerMove)
  canvas.value.removeEventListener('pointerdown', onPointerDown)
  canvas.value.removeEventListener('click', onClick)
  controls?.dispose()
  // 释放 GPU 资源,避免反复切换页面内存泄漏
  if (model) {
    model.traverse((o) => {
      if (o.isMesh) {
        o.geometry?.dispose()
        const mats = Array.isArray(o.material) ? o.material : [o.material]
        mats.forEach((m) => m?.dispose())
      }
    })
  }
  if (floor) {
    floor.geometry?.dispose()
    floor.material?.map?.dispose()
    floor.material?.dispose()
  }
  rgbLights.forEach((l) => l.dispose())
  renderer?.dispose()
})

// 暴露给父组件调用(如换屏幕贴图)
defineExpose({ setScreenTexture })
</script>

<style scoped>
.bedroom-model {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
}
.bedroom-canvas {
  display: block;
  width: 100%;
  height: 100%;
}
.loading {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  color: #999;
  font-size: 14px;
  letter-spacing: 3px;
  pointer-events: none;
}
</style>
