<template>
  <div class="rubiks-container">
    <div class="control-panel" :class="{ isDark: !isDark }">
      <h3>魔方</h3>

      <div class="mode-select">
        <button
          :class="mode === 'game' ? 'active' : ''"
          @click="switchMode('game')"

        >
          游戏模式
        </button>
        <button
          :class="mode === 'show' ? 'active' : ''"
          @click="switchMode('show')"
        >
          展示模式
        </button>
      </div>

      <div class="axis-select">
        <label>选择轴：</label>
        <button 
          v-for="axis in ['x','y','z']" 
          :key="axis"
          :class="activeAxis === axis ? 'active' : ''"
          @click="switchAxis(axis)"
          :disabled="mode === 'show'"
        >
          {{ axis.toUpperCase() }}轴
        </button>
      </div>
      <div class="layer-select">
        <label>选择层级：</label>
        <button 
          v-for="layer in [-1,0,1]" 
          :key="layer"
          :class="activeLayer === layer ? 'active' : ''"
          @click="switchLayer(layer)"
          :disabled="mode === 'show'"
        >
          {{ layer === -1 ? '左/底/后' : layer === 0 ? '中' : '右/顶/前' }}
        </button>
      </div>
      <div class="rotate-btns">
        <button @click="rotateLayer(true);setAngle(0)"  :disabled="mode === 'show' ">顺时针旋转 90°</button>
        <button @click="rotateLayer(false);setAngle(90)" :disabled="mode === 'show'">逆时针旋转 90°</button>
        <button class="reset-btn" @click="resetCube" :disabled="mode === 'show'">重置魔方</button>
      </div>
      <div class="lock-btn-container">
        <button 
          class="lock-btn" 
          @click="togglePageLock"
        >
          {{ pageLocked ? "已锁定滚动" : "允许滚动" }}
        </button>
      </div>
    </div>
    <canvas ref="canvasContainer" class="cube-canvas"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, defineProps } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

const setAngle = async (angle) => {
  await fetch("http://192.168.194.130/set?angle=" + angle).catch(() => {});
};

const { isDark, pageLocked } = defineProps({
  isDark: { type: Boolean, default: true },
  pageLocked: { type: Boolean, default: false }
})
const emit = defineEmits(['toggle-page-lock'])

const canvasContainer = ref(null)

let scene, camera, renderer, controls
let cubeGroup = new THREE.Group()
let cubies = []
let isRotating = false

const activeAxis = ref('y')
const activeLayer = ref(0)
const mode = ref('show')

const COLORS = {
  U: 0xFFFFFF, D: 0xFFFF00, F: 0xFF0000,
  B: 0xFFA500, L: 0x0000FF, R: 0x00FF00
}

function initScene() {
  scene = new THREE.Scene()
  scene.background = new THREE.Color(isDark ? 0x000000 : 0xfaf7f0)

  camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(8, 3, 8)
  camera.lookAt(0, 0, 0)
  
  renderer = new THREE.WebGLRenderer({ antialias: true, canvas: canvasContainer.value })
  renderer.setSize(window.innerWidth, window.innerHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2))
  
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.6)
  scene.add(ambientLight)
  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8)
  directionalLight.position.set(10, 10, 10)
  scene.add(directionalLight)
  
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
  controls.minDistance = 5
  controls.maxDistance = 12
  controls.enableZoom = false
  
  scene.add(cubeGroup)
}

watch(() => isDark, (val) => {
  if (scene) scene.background = new THREE.Color(val ? 0x000000 : 0xfaf7f0)
})

function createCubie(x, y, z) {
  const size = 0.95
  const geo = new THREE.BoxGeometry(size, size, size)
  const black = new THREE.MeshLambertMaterial({ color: 0x111111 })

  const mats = [
    black.clone(), black.clone(),
    black.clone(), black.clone(),
    black.clone(), black.clone()
  ]

  if (x === 1) mats[0].color.setHex(COLORS.R)
  if (x === -1) mats[1].color.setHex(COLORS.L)
  if (y === 1) mats[2].color.setHex(COLORS.U)
  if (y === -1) mats[3].color.setHex(COLORS.D)
  if (z === 1) mats[4].color.setHex(COLORS.F)
  if (z === -1) mats[5].color.setHex(COLORS.B)

  const mesh = new THREE.Mesh(geo, mats)
  mesh.position.set(x, y, z)
  mesh.userData.coord = { x, y, z }

  const edges = new THREE.EdgesGeometry(geo)
  const border = new THREE.LineSegments(edges, new THREE.LineBasicMaterial({ color: 0x000000 }))
  mesh.add(border)
  mesh.userData.border = border

  return mesh
}

function initRubiksCube() {
  cubeGroup.clear()
  cubies = []
  for (let x = -1; x <= 1; x++) {
    for (let y = -1; y <= 1; y++) {
      for (let z = -1; z <= 1; z++) {
        const c = createCubie(x, y, z)
        cubies.push(c)
        cubeGroup.add(c)
      }
    }
  }
  updateHighlight()
}

function switchAxis(axis) {
  if (isRotating || mode.value === 'show') return
  activeAxis.value = axis
  updateHighlight()
}
function switchLayer(layer) {
  if (isRotating || mode.value === 'show') return
  activeLayer.value = layer
  updateHighlight()
}

function updateHighlight() {
  if (mode.value === 'show' || isRotating) return
  const a = activeAxis.value
  const l = activeLayer.value

  cubies.forEach(c => {
    const cx = Math.round(c.position.x)
    const cy = Math.round(c.position.y)
    const cz = Math.round(c.position.z)
    let sel = false
    if (a === 'x' && cx === l) sel = true
    if (a === 'y' && cy === l) sel = true
    if (a === 'z' && cz === l) sel = true
    c.userData.border.material.color.setHex(sel ? 0xffffff : 0x000000)
  })
}

// ==========================================
// 核心：完美旋转（中心自转 + 外围公转，颜色不乱）
// ==========================================
function rotateLayer(clockwise) {
  if (isRotating || mode.value === 'show') return
  isRotating = true

  const axis = activeAxis.value
  const layer = activeLayer.value
  const angle = Math.PI / 2
  const dir = clockwise ? 1 : -1

  // 找到需要旋转的小方块
  const targets = cubies.filter(c => {
    const cx = Math.round(c.position.x)
    const cy = Math.round(c.position.y)
    const cz = Math.round(c.position.z)
    if (axis === 'x' && cx === layer) return true
    if (axis === 'y' && cy === layer) return true
    if (axis === 'z' && cz === layer) return true
    return false
  })

  // 找到旋转中心点
  const center = new THREE.Vector3(
    axis === 'x' ? layer : 0,
    axis === 'y' ? layer : 0,
    axis === 'z' ? layer : 0
  )

  // 记录每个方块的初始旋转四元数和位置
  targets.forEach(c => {
    c.userData.startPosition = c.position.clone()
    c.userData.startQuaternion = c.quaternion.clone()
  })

  let rotated = 0
  const speed = 0.08

  function animateRot() {
    const step = Math.min(speed, angle - rotated)
    rotated += step

    // 手动计算每个方块的新位置，不使用pivot进行旋转（避免自转）
    targets.forEach(c => {
      // 获取相对中心的位置
      const relPos = c.userData.startPosition.clone().sub(center)
      let newPos = relPos.clone()

      // 根据轴旋转位置
      if (axis === 'x') {
        // 绕X轴旋转 - 只影响y和z坐标
        const y = relPos.y
        const z = relPos.z
        // 修正X轴旋转方向
        newPos.y = y * Math.cos(rotated * dir) - z * Math.sin(rotated * dir)
        newPos.z = y * Math.sin(rotated * dir) + z * Math.cos(rotated * dir)
      } else if (axis === 'y') {
        // 绕Y轴旋转 - 只影响x和z坐标
        const x = relPos.x
        const z = relPos.z
        newPos.x = x * Math.cos(rotated * dir) + z * Math.sin(rotated * dir)
        newPos.z = -x * Math.sin(rotated * dir) + z * Math.cos(rotated * dir)
      } else if (axis === 'z') {
        // 绕Z轴旋转 - 只影响x和y坐标
        const x = relPos.x
        const y = relPos.y
        newPos.x = x * Math.cos(rotated * dir) - y * Math.sin(rotated * dir)
        newPos.y = x * Math.sin(rotated * dir) + y * Math.cos(rotated * dir)
      }

      // 更新位置（加回中心点偏移）
      c.position.copy(newPos.add(center))

      // 不改变方块的自身旋转，只改变位置
      c.quaternion.copy(c.userData.startQuaternion)
    })

    if (rotated < angle) {
      requestAnimationFrame(animateRot)
    } else {
      // 完成旋转，确保位置规整
      targets.forEach(c => {
        c.position.x = Math.round(c.position.x)
        c.position.y = Math.round(c.position.y)
        c.position.z = Math.round(c.position.z)

        // 更新面的颜色顺序
        updateFace(c, axis, clockwise)

        // 清理临时数据
        delete c.userData.startPosition
        delete c.userData.startQuaternion
      })

      isRotating = false
      updateHighlight()
    }
  }

  animateRot()
}

// 正确更新颜色面
function updateFace(cubie, axis, cw) {
  const tmp = [...cubie.material]
  const result = [...tmp]

  if (axis === 'y') {
    // Y 轴（你原来的是对的）
    if (cw) {
      result[0] = tmp[4]; result[1] = tmp[5]; result[4] = tmp[1]; result[5] = tmp[0]
    } else {
      result[0] = tmp[5]; result[1] = tmp[4]; result[4] = tmp[0]; result[5] = tmp[1]
    }
  } 
  // ====================== 重点修复：X 轴 ======================
  else if (axis === 'x') {
    if (cw) {
      // 顺时针 X 轴：上 → 前 → 下 → 后 → 上 正确轮换
      result[2] = tmp[5] // 上 = 原来的后
      result[3] = tmp[4] // 下 = 原来的前
      result[4] = tmp[2] // 前 = 原来的上
      result[5] = tmp[3] // 后 = 原来的下
    } else {
      // 逆时针 X 轴
      result[2] = tmp[4]
      result[3] = tmp[5]
      result[4] = tmp[3]
      result[5] = tmp[2]
    }
  }
  // ==========================================================
  else if (axis === 'z') {
    // Z 轴（你原来的是对的）
    if (cw) {
      result[0] = tmp[3]; result[1] = tmp[2]; result[2] = tmp[0]; result[3] = tmp[1]
    } else {
      result[0] = tmp[2]; result[1] = tmp[3]; result[2] = tmp[1]; result[3] = tmp[0]
    }
  }

  cubie.material = result
}

function resetCube() {
  if (isRotating || mode.value === 'show') return
  initRubiksCube()
  activeAxis.value = 'y'
  activeLayer.value = 0
}

function switchMode(m) {
  if (isRotating) return
  mode.value = m
  controls.enabled = m === 'game'
}

// 锁定滚动
function togglePageLock() {
  emit('toggle-page-lock', !pageLocked)
}

// ==========================================
// 修复爆栈 animate 函数！！！
// ==========================================
function animate() {
  requestAnimationFrame(animate) // ✅ 正确写法
  controls.update()
  if (mode.value === 'show') {
    cubeGroup.rotation.y += 0.003
  }
  renderer.render(scene, camera)
}

function handleResize() {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth, window.innerHeight)
}

onMounted(() => {
  initScene()
  initRubiksCube()
  animate()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  renderer.dispose()
})
</script>

<style scoped>
* { margin: 0; padding: 0; box-sizing: border-box; }
.rubiks-container { width: 100vw; height: 100vh; position: relative; overflow: hidden; }
.cube-canvas { display: block; width: 100%; height: 100%; position: absolute; top: 0; left: 0; }

.control-panel {
  position: absolute; bottom: 20px; left: 50%; transform: translateX(-50%);
  background: rgba(0, 0, 0, 0.8);
  color: #ffffff;
  z-index: 100;
  min-width: 380px;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0,0,0,0.5);
  transition: all 0.3s ease;
}
.control-panel.isDark {
  background: rgba(250, 247, 240, 0.95);
  color: #111111;
  box-shadow: 0 0 10px rgba(0,0,0,0.1);
}

.control-panel h3 { text-align: center; margin-bottom: 12px; }
.mode-select, .axis-select, .layer-select, .rotate-btns {
  margin: 12px 0; display: flex; gap: 8px; flex-wrap: wrap; justify-content: center;
}
.lock-btn-container { margin-top: 10px; display: flex; justify-content: center; }
.lock-btn { padding: 8px 16px; background: #17a2b8 !important; color: #fff !important; font-weight: bold; width: 100%; }
.lock-btn:hover { background: #138496 !important; }

button {
  padding: 7px 14px;
  border: 1px solid #1aa2a9;  /* 白色边框 = 镂空感 */
  border-radius: 5px;
  cursor: pointer;
  background: transparent; /* 透明底 */
  color: white;
  transition: all 0.25s ease;
}

/* 悬浮：半透明填充 */
button:hover {
  background: rgba(255, 255, 255, 0.1);
}

/* 激活：实心高亮 */
button.active {
  background: #023a52;
  border-color: #007bff;
  color: #fff;
  font-weight: bold;
}

button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

/* 亮色模式下的镂空样式 */
.control-panel.isDark button {
  border-color: #333;
  background: transparent;
  color: #111;
}
.control-panel.isDark button:hover {
  background: rgba(0, 0, 0, 0.05);
}
.control-panel.isDark button.active {
  background: #034285;
  border-color: #073b74;
  color: #fff;
}

/* 重置按钮特殊样式 */
.reset-btn {
  background: #64030c !important;
  border-color: #dc3545 !important;
  font-weight: bold;
  color: #fff !important;
}
</style>