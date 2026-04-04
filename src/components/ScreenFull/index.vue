<template>
  <div>
    <!-- 全屏切换按钮 - 使用图片 -->
    <a 
      class="nav-link platform-link icon-btn" 
      @click="onToggle"
      target="_self"
    >
      <!-- Vite 动态图片引入方式 -->
      <img 
        :src="isFullscreen ? exitIcon : screenIcon" 
        class="icon-img" 
        alt="全屏切换"
      />
    </a>
  </div>
</template>
 
<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import screenfull from 'screenfull'

// Vite 正确引入静态图片
const screenIcon = new URL('@/assets/screenfull.png', import.meta.url).href
const exitIcon = new URL('@/assets/screenfullexit.png', import.meta.url).href

// 是否全屏
const isFullscreen = ref(false)

// 监听变化
const change = () => {
  isFullscreen.value = screenfull.isFullscreen
}

// 切换事件
const onToggle = () => {
  screenfull.toggle()
}

// 设置侦听器
onMounted(() => {
  screenfull.on('change', change)
})

// 删除侦听器
onUnmounted(() => {
  screenfull.off('change', change)
})
</script>

<style scoped>
.nav-link {
  font-size: 1rem;
  font-weight: 500;
  text-decoration: none;
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.4rem;
  transition: transform 0.3s ease;
  cursor: pointer;
}
.nav-link:hover {
  transform: scale(1.2);
}

.theme-light .nav-link {
  color: #222;
}

.icon-img {
  width: 30px;
  height: 30px;
  object-fit: contain;
}

.icon-btn {
  padding: 0 !important;
  border: none !important;
  background: transparent !important;
}
</style>