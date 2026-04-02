<template>
  <div class="rubiks-container">
    <div class="control-panel">
      <h3>3D魔方控制器</h3>
      <div class="axis-select">
        <label>选择轴：</label>
        <button 
          v-for="axis in ['x','y','z']" 
          :key="axis"
          :class="activeAxis === axis ? 'active' : ''"
          @click="switchAxis(axis)"
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
        >
          {{ layer === -1 ? '左/底/后' : layer === 0 ? '中' : '右/顶/前' }}
        </button>
      </div>
      <div class="rotate-btns">
        <button @click="rotateLayer(true)">顺时针旋转 90°</button>
        <button @click="rotateLayer(false)">逆时针旋转 90°</button>
        <button class="reset-btn" @click="resetCube">🔄 重置魔方</button>
      </div>
    </div>
    <canvas ref="canvasContainer" class="cube-canvas"></canvas>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as THREE from 'three'
import { OrbitControls } from 'three/addons/controls/OrbitControls.js'

const canvasContainer = ref(null)

let scene, camera, renderer, controls
let cubeGroup = new THREE.Group()
let cubies = []
let isRotating = false

const activeAxis = ref('y')
const activeLayer = ref(0)

const COLORS = {
  U: 0xFFFFFF, // 上层 - 白色
  D: 0xFFFF00, // 下层 - 黄色
  F: 0xFF0000, // 正面 - 红色
  B: 0xFFA500, // 背面 - 橙色
  L: 0x0000FF, // 左面 - 蓝色
  R: 0x00FF00  // 右面 - 绿色
};

// 世界坐标工具（永不改变）
const worldPos = new THREE.Vector3()

function initScene() {
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x1a1a2e)
  
  camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000)
  camera.position.set(8, 8, 8)
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
  
  scene.add(cubeGroup)
}

function createCubie(x, y, z) {
  const size = 0.95
  const geometry = new THREE.BoxGeometry(size, size, size)
  
  const materials = [
    new THREE.MeshLambertMaterial({ color: COLORS.R }),
    new THREE.MeshLambertMaterial({ color: COLORS.L }),
    new THREE.MeshLambertMaterial({ color: COLORS.U }),
    new THREE.MeshLambertMaterial({ color: COLORS.D }),
    new THREE.MeshLambertMaterial({ color: COLORS.F }),
    new THREE.MeshLambertMaterial({ color: COLORS.B })
  ]
  
  const cube = new THREE.Mesh(geometry, materials)
  cube.position.set(x, y, z)
  cube.userData.originalPos = new THREE.Vector3(x, y, z)
  cube.userData.isHighlighted = false
  
  const edges = new THREE.EdgesGeometry(geometry)
  const border = new THREE.LineSegments(edges, new THREE.LineBasicMaterial({ color: 0xffff00 }))
  border.visible = false
  cube.add(border)
  cube.userData.border = border
  
  return cube
}

function initRubiksCube() {
  cubies = []
  for (let x = -1; x <= 1; x++) {
    for (let y = -1; y <= 1; y++) {
      for (let z = -1; z <= 1; z++) {
        const cubie = createCubie(x, y, z)
        cubies.push(cubie)
        cubeGroup.add(cubie)
      }
    }
  }
  updateHighlight()
}

function switchAxis(axis) {
  if (isRotating) return
  activeAxis.value = axis
  updateHighlight()
}

function switchLayer(layer) {
  if (isRotating) return
  activeLayer.value = layer
  updateHighlight()
}

// ==================== ✅ 修复核心：使用真实世界坐标判断 ====================
function updateHighlight() {
  const axis = activeAxis.value
  const layer = activeLayer.value
  
  cubies.forEach(cubie => {
    // 获取小方块在世界中的真实坐标（不受父级旋转影响）
    cubie.getWorldPosition(worldPos)
    
    let isSelected = false
    // 四舍五入解决浮点误差
    const wx = Math.round(worldPos.x)
    const wy = Math.round(worldPos.y)
    const wz = Math.round(worldPos.z)

    if (axis === 'x' && wx === layer) isSelected = true
    if (axis === 'y' && wy === layer) isSelected = true
    if (axis === 'z' && wz === layer) isSelected = true
    
    cubie.userData.isHighlighted = isSelected
    cubie.userData.border.visible = isSelected
  })
}

function rotateLayer(clockwise) {
  if (isRotating) return
  isRotating = true
  
  const axis = activeAxis.value
  const layer = activeLayer.value
  const targetAngle = clockwise ? -Math.PI / 2 : Math.PI / 2
  
  // ==================== ✅ 修复核心：旋转前也用世界坐标筛选 ====================
  const targetCubies = cubies.filter(cubie => {
    cubie.getWorldPosition(worldPos)
    const wx = Math.round(worldPos.x)
    const wy = Math.round(worldPos.y)
    const wz = Math.round(worldPos.z)

    if (axis === 'x') return wx === layer
    if (axis === 'y') return wy === layer
    if (axis === 'z') return wz === layer
  })

  const pivot = new THREE.Group()
  scene.add(pivot)

  targetCubies.forEach(cubie => {
    pivot.attach(cubie)
  })

  let currentAngle = 0
  const animateRotate = () => {
    const step = targetAngle * 0.1
    currentAngle += step
    
    if (axis === 'x') pivot.rotation.x = currentAngle
    if (axis === 'y') pivot.rotation.y = currentAngle
    if (axis === 'z') pivot.rotation.z = currentAngle
    
    if (Math.abs(currentAngle) < Math.abs(targetAngle)) {
      requestAnimationFrame(animateRotate)
    } else {
      finishRotation(targetCubies, pivot)
    }
  }
  animateRotate()
}

function finishRotation(targetCubies, pivot) {
  targetCubies.forEach(cubie => {
    cubeGroup.attach(cubie)
    cubie.updateMatrixWorld(true)
  })

  scene.remove(pivot)
  updateHighlight()
  isRotating = false
}

function resetCube() {
  if (isRotating) return
  cubies.forEach(cubie => {
    cubie.position.copy(cubie.userData.originalPos)
    cubie.rotation.set(0, 0, 0)
    cubie.updateMatrixWorld(true)
  })
  activeAxis.value = 'y'
  activeLayer.value = 0
  updateHighlight()
}

function animate() {
  requestAnimationFrame(animate)
  controls.update()
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
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.rubiks-container {
  width: 100vw;
  height: 100vh;
  position: relative;
  overflow: hidden;
}
.cube-canvas {
  display: block;
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
}
.control-panel {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(0,0,0,0.75);
  padding: 20px;
  border-radius: 10px;
  color: white;
  z-index: 100;
  min-width: 380px;
  box-shadow: 0 0 10px rgba(0,0,0,0.5);
}
.control-panel h3 {
  margin-top: 0;
  text-align: center;
  color: #fff;
}
.axis-select, .layer-select, .rotate-btns {
  margin: 15px 0;
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  justify-content: center;
}
button {
  padding: 7px 14px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  background: #444;
  color: white;
  transition: all 0.2s ease;
  font-size: 14px;
}
button:hover {
  background: #666;
  transform: translateY(-1px);
}
button.active {
  background: #007bff;
  font-weight: bold;
}
.reset-btn {
  background: #dc3545;
  font-weight: bold;
}
</style>