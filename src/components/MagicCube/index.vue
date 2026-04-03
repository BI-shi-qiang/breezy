<template>
  <div class="rubiks-container">
    <div class="control-panel">
      <h3>魔方</h3>

      <!-- 新增：模式切换 -->
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
        <button @click="rotateLayer(true)" :disabled="mode === 'show'">顺时针旋转 90°</button>
        <button @click="rotateLayer(false)" :disabled="mode === 'show'">逆时针旋转 90°</button>
        <button class="reset-btn" @click="resetCube" :disabled="mode === 'show'">重置魔方</button>
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

// 原有状态
const activeAxis = ref('y')
const activeLayer = ref(0)

// 新增：模式
const mode = ref('show')
const autoRotateSpeed = 0.003
const blendSpeed = 0.1

const COLORS = {
  U: 0xFFFFFF, D: 0xFFFF00, F: 0xFF0000,
  B: 0xFFA500, L: 0x0000FF, R: 0x00FF00
};

const worldPos = new THREE.Vector3()

// 初始化场景
function initScene() {
  scene = new THREE.Scene()
  scene.background = new THREE.Color(0x000000)
  
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
  
  controls.minDistance = 5
  controls.maxDistance = 12

  scene.add(cubeGroup)
}

// 创建小方块
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

// 初始化魔方
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

// 切换轴/层级
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

// 高亮
function updateHighlight() {
  // 如果是展示模式或者正在旋转，直接返回（不处理高亮）
  if (mode.value === 'show' || isRotating) return;

  const axis = activeAxis.value;
  const layer = activeLayer.value; // 注意：这里原来是 activeAxis，我修正为 activeLayer

  cubies.forEach(cubie => {
    cubie.getWorldPosition(worldPos);
    let isSelected = false;
    
    // 获取当前方块在世界坐标系中的整数位置
    const wx = Math.round(worldPos.x);
    const wy = Math.round(worldPos.y);
    const wz = Math.round(worldPos.z);

    // 判断是否属于当前选中的层
    if (axis === 'x' && wx === layer) isSelected = true;
    if (axis === 'y' && wy === layer) isSelected = true;
    if (axis === 'z' && wz === layer) isSelected = true;

    // ✅ 核心逻辑：选中变白，未选中变黑
    cubie.userData.border.material.color.set(isSelected ? 0xffffff : 0x000000);
  });
}

// 旋转层
function rotateLayer(clockwise) {
  if (isRotating || mode.value === 'show') return
  isRotating = true
  
  const axis = activeAxis.value
  const layer = activeLayer.value
  const targetAngle = clockwise ? -Math.PI / 2 : Math.PI / 2
  
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
  targetCubies.forEach(cubie => pivot.attach(cubie))

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

// 重置
function resetCube() {
  if (isRotating || mode.value === 'show') return
  cubies.forEach(cubie => {
    cubie.position.copy(cubie.userData.originalPos)
    cubie.rotation.set(0, 0, 0)
    cubie.updateMatrixWorld(true)
  })
  activeAxis.value = 'y'
  activeLayer.value = 0
  updateHighlight()
}

function switchMode(newMode) {
  if (isRotating) return
  mode.value = newMode

  if (newMode === 'show') {
    controls.enabled = false
  } else {
    controls.enabled = true
  }
}

function updateShowMode() {
  const isShow = mode.value === 'show';

  if (isShow) {
    cubeGroup.rotation.y += autoRotateSpeed;
  } else {
    cubeGroup.rotation.y *= 0.9; // 阻尼回归
  }

  cubies.forEach(c => {
    const ox = c.userData.originalPos.x;
    const oy = c.userData.originalPos.y;
    const oz = c.userData.originalPos.z;

    let targetX, targetY, targetZ;
    if (isShow) {
      targetX = ox + (ox === 0 ? 0 : ox * 1.3);
      targetY = oy + (oy === 0 ? 0 : oy * 1.3);
      targetZ = oz + (oz === 0 ? 0 : oz * 1.3);
    } else {
      targetX = ox;
      targetY = oy;
      targetZ = oz;
    }
    c.position.lerp(new THREE.Vector3(targetX, targetY, targetZ), blendSpeed);

    c.material.forEach(m => {
      m.emissive.set(m.color);
      m.emissiveIntensity = isShow ? 0.8 : m.emissiveIntensity * 0.9;
    });

    const border = c.userData.border;
    
    if (isShow) {
      border.visible = true;
      border.material.color.set(0xffffff);
    } else {
      border.visible = true; 
    }
  });
}

// 主渲染
function animate() {
  requestAnimationFrame(animate)
  controls.update()
  updateShowMode()
  renderer.render(scene, camera)
}

// 窗口大小
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
  background: rgba(0,0,0,0.75); padding: 20px; border-radius: 10px;
  color: white; z-index: 100; min-width: 380px;
  box-shadow: 0 0 10px rgba(0,0,0,0.5);
}
.control-panel h3 { text-align: center; margin-bottom: 12px; }

.mode-select {
  margin: 0 0 15px 0; display: flex; gap: 8px; justify-content: center;
}
.axis-select, .layer-select, .rotate-btns {
  margin: 15px 0; display: flex; gap: 8px; flex-wrap: wrap; justify-content: center;
}

button {
  padding: 7px 14px; border: none; border-radius: 5px; cursor: pointer;
  background: #444; color: white; transition: all 0.2s ease; font-size: 14px;
}
button:hover { background: #666; transform: translateY(-1px); }
button.active { background: #007bff; font-weight: bold; }
button:disabled { opacity: 0.4; cursor: not-allowed; }

.reset-btn { background: #dc3545; font-weight: bold; }
</style>