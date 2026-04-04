<template>
  <div class="webgl-background" ref="containerRef">
    <canvas ref="canvasRef" class="webgl-canvas" />
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch } from 'vue'
import * as THREE from 'three'

const containerRef = ref(null)
const canvasRef = ref(null)

let scene, camera, renderer, points, animationId
let mouseX = 0, mouseY = 0

const initScene = () => {
  const container = containerRef.value
  const { clientWidth, clientHeight } = container

  scene = new THREE.Scene()
  camera = new THREE.PerspectiveCamera(60, clientWidth / clientHeight, 0.1, 1000)
  camera.position.z = 30

  renderer = new THREE.WebGLRenderer({
    canvas: canvasRef.value,
    alpha: true,
    antialias: true,
    powerPreference: 'high-performance'
  })
  renderer.setSize(clientWidth, clientHeight)
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 1.5))

  const particlesGeometry = new THREE.BufferGeometry()
  const count = 1000
  const positions = new Float32Array(count * 3)

  for (let i = 0; i < count * 3; i++) {
    positions[i] = (Math.random() - 0.5) * 70
  }
  particlesGeometry.setAttribute('position', new THREE.BufferAttribute(positions, 3))

  const particlesMaterial = new THREE.PointsMaterial({
    color: 0x4FD1FF,
    size: 0.15,
    transparent: true,
    opacity: 0.7
  })

  points = new THREE.Points(particlesGeometry, particlesMaterial)
  scene.add(points)

  window.addEventListener('mousemove', onMouseMove)
  window.addEventListener('resize', onResize)
}

const onMouseMove = (e) => {
  mouseX = (e.clientX / window.innerWidth) - 0.5
  mouseY = (e.clientY / window.innerHeight) - 0.5
}

const onResize = () => {
  const { clientWidth, clientHeight } = containerRef.value
  camera.aspect = clientWidth / clientHeight
  camera.updateProjectionMatrix()
  renderer.setSize(clientWidth, clientHeight)
}

const renderLoop = () => {
  if (!points) return
  points.rotation.x += 0.0012
  points.rotation.y += 0.001
  points.position.x += (mouseX * 2 - points.position.x) * 0.05
  points.position.y += (-mouseY * 2 - points.position.y) * 0.05
  renderer.render(scene, camera)
  animationId = requestAnimationFrame(renderLoop)
}

onMounted(() => {
  initScene()
  renderLoop()
})

onUnmounted(() => {
  cancelAnimationFrame(animationId)
  window.removeEventListener('mousemove', onMouseMove)
  window.removeEventListener('resize', onResize)
  renderer?.dispose()
})
</script>

<style scoped>
.webgl-background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 3; /* 放在最底层 */
  pointer-events: none; /* 不影响点击、滑动 */
  overflow: hidden;
}

.webgl-canvas {
  width: 100%;
  height: 100%;
  display: block;
}
</style>