<template>
  <div class="rubiks-cube-container" ref="containerRef">
    <div class="controls">
      <button @click="switchMode('row')">行选择模式(Y轴层)</button>
      <button @click="switchMode('col')">列选择模式(X轴层)</button>
      <button @click="rotateLayer(true)">顺时针旋转90°</button>
      <button @click="rotateLayer(false)">逆时针旋转90°</button>
      <!-- 新增：重置按钮 -->
      <button @click="resetCube" style="background:#ff4444;">重置魔方</button>
      <p>当前模式：{{ selectMode }} | 选中层：{{ selectedLayer }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

const containerRef = ref(null)

let scene, camera, renderer, controls
let cubeGroup = null
let pivot = null
let cubies = []

const selectMode = ref('row')
const selectedLayer = ref(1)

const COLORS = {
  U: 0xffffff,
  D: 0xffd500,
  F: 0x00ff00,
  B: 0x0000ff,
  L: 0xff0000,
  R: 0xffa500
}

function initScene() {
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x1a1a2e)
}

function initCamera() {
  camera = new THREE.PerspectiveCamera(60, containerRef.value.clientWidth / containerRef.value.clientHeight, 0.1, 1000)
  camera.position.set(8, 8, 10)
  camera.lookAt(0, 0, 0)
}

function initRenderer() {
  renderer = new THREE.WebGLRenderer({ antialias: true })
  renderer.setSize(containerRef.value.clientWidth, containerRef.value.clientHeight)
  renderer.shadowMap.enabled = true
  containerRef.value.appendChild(renderer.domElement)
}

function initControls() {
  controls = new OrbitControls(camera, renderer.domElement)
  controls.enableDamping = true
  controls.dampingFactor = 0.05
}

function initLight() {
  const ambient = new THREE.AmbientLight(0xffffff, 0.6)
  scene.add(ambient)
  const directional = new THREE.DirectionalLight(0xffffff, 0.8)
  directional.position.set(10, 10, 10)
  scene.add(directional)
}

// 创建小方块
function createCubie(x, y, z) {
  const size = 1
  const gap = 0.03
  const geometry = new THREE.BoxGeometry(size - gap, size - gap, size - gap)

  const materials = [
    new THREE.MeshLambertMaterial({ color: COLORS.R }),
    new THREE.MeshLambertMaterial({ color: COLORS.L }),
    new THREE.MeshLambertMaterial({ color: COLORS.U }),
    new THREE.MeshLambertMaterial({ color: COLORS.D }),
    new THREE.MeshLambertMaterial({ color: COLORS.F }),
    new THREE.MeshLambertMaterial({ color: COLORS.B })
  ]

  const mesh = new THREE.Mesh(geometry, materials)
  mesh.position.set(x, y, z)

  mesh.userData = {
    originalX: x,
    originalY: y,
    originalZ: z,
    currentX: x,
    currentY: y,
    currentZ: z,
    isHighlighted: false
  }

  const edges = new THREE.EdgesGeometry(geometry)
  const border = new THREE.LineSegments(edges, new THREE.LineBasicMaterial({ color: 0x000000, linewidth: 2 }))
  mesh.add(border)

  return mesh
}

// 初始化魔方
function initRubiksCube() {
  cubeGroup = new THREE.Group()
  scene.add(cubeGroup)
  cubies = [] // 清空旧方块

  for (let x = -1; x <= 1; x++) {
    for (let y = -1; y <= 1; y++) {
      for (let z = -1; z <= 1; z++) {
        const cubie = createCubie(x, y, z)
        cubies.push(cubie)
        cubeGroup.add(cubie)
      }
    }
  }

  updateHighlightLayer()
}

// 高亮选中层
function updateHighlightLayer() {
  const target = selectedLayer.value - 1

  cubies.forEach(cubie => {
    let isSelected = false
    if (selectMode.value === 'row') {
      isSelected = cubie.userData.currentY === target
    } else {
      isSelected = cubie.userData.currentX === target
    }
    cubie.children[0].material.color.set(isSelected ? 0xffff00 : 0x000000)
    cubie.userData.isHighlighted = isSelected
  })
}

function getSelectedCubies() {
  return cubies.filter(c => c.userData.isHighlighted)
}

// 创建旋转轴心
function createPivot(selectedCubies) {
  if (pivot) {
    scene.remove(pivot)
    pivot = null
  }
  pivot = new THREE.Group()
  scene.add(pivot)
  selectedCubies.forEach(cubie => pivot.attach(cubie))
}

// 旋转
function rotateLayer(clockwise = true) {
  const selectedCubies = getSelectedCubies()
  if (selectedCubies.length === 0) return

  createPivot(selectedCubies)

  const targetAngle = clockwise ? -Math.PI / 2 : Math.PI / 2
  const rotateAxis = selectMode.value === 'row' ? 'Y' : 'X'
  const duration = 600
  let startTime = null

  function animate(timestamp) {
    if (!startTime) startTime = timestamp
    const progress = Math.min((timestamp - startTime) / duration, 1)
    const easeProgress = 0.5 - 0.5 * Math.cos(progress * Math.PI)

    if (rotateAxis === 'Y') {
      pivot.rotation.y = easeProgress * targetAngle
    } else {
      pivot.rotation.x = easeProgress * targetAngle
    }

    if (progress < 1) {
      requestAnimationFrame(animate)
    } else {
      bakeTransform(selectedCubies)
      // ========== 关键：旋转结束后自动保持选中 ==========
      updateHighlightLayer()
    }
  }

  requestAnimationFrame(animate)
}

// 坐标烘焙
function bakeTransform(selectedCubies) {
  selectedCubies.forEach(cubie => {
    cubeGroup.attach(cubie)
    const mat = new THREE.Matrix4().copy(cubie.matrixWorld)
    const pos = new THREE.Vector3()
    mat.decompose(pos, new THREE.Quaternion(), new THREE.Vector3())

    cubie.userData.currentX = Math.round(pos.x)
    cubie.userData.currentY = Math.round(pos.y)
    cubie.userData.currentZ = Math.round(pos.z)
  })

  scene.remove(pivot)
  pivot = null
}

// ========== 新增：重置魔方 ==========
function resetCube() {
  // 清除正在旋转的动画
  if (pivot) {
    scene.remove(pivot)
    pivot = null
  }
  // 销毁旧魔方，重建全新的
  scene.remove(cubeGroup)
  initRubiksCube()
}

function switchMode(mode) {
  selectMode.value = mode
  updateHighlightLayer()
}

function changeLayer() {
  selectedLayer.value = (selectedLayer.value + 1) % 3
  updateHighlightLayer()
}

function animate() {
  requestAnimationFrame(animate)
  controls.update()
  renderer.render(scene, camera)
}

function handleResize() {
  camera.aspect = containerRef.value.clientWidth / containerRef.value.clientHeight
  camera.updateProjectionMatrix()
  renderer.setSize(containerRef.value.clientWidth, containerRef.value.clientHeight)
}

onMounted(() => {
  initScene()
  initCamera()
  initRenderer()
  initControls()
  initLight()
  initRubiksCube()
  animate()
  window.addEventListener('resize', handleResize)
  containerRef.value.addEventListener('click', changeLayer)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  renderer.dispose()
})
</script>

<style scoped>
.rubiks-cube-container {
  width: 100%;
  height: 85vh;
  position: relative;
  background: #0f0f1a;
}

.controls {
  position: absolute;
  left: 50%;
  bottom: 30px;
  transform: translateX(-50%);
  z-index: 10;
  background: rgba(0,0,0,0.7);
  padding: 12px;
  border-radius: 8px;
  color: white;
  text-align: center;
}

button {
  margin: 4px;
  padding: 6px 12px;
  border: none;
  border-radius: 4px;
  background: #4e6aff;
  color: white;
  cursor: pointer;
}

button:hover {
  background: #3d5aff;
}

p {
  margin: 8px 0 0;
  font-size: 14px;
}
</style>