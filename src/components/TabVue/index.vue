<template>
  <nav class="navbar">
    <div class="nav-container">
      <a href="#" class="nav-brand">Breezy</a>
      <div class="nav-menu">
        <a href="https://github.com/BI-shi-qiang" target="_blank" class="nav-link platform-link">
          <img src="@/assets/github.png" class="icon-img" />
        </a>
        <a href="https://gitee.com/bi-shiqiang" target="_blank" class="nav-link platform-link">
          <img src="@/assets/gitee.png" class="icon-img" />
        </a>
        <a href="https://blog.csdn.net/2401_82352441?type=blog" target="_blank" class="nav-link platform-link">
          <img src="@/assets/csdn.png" class="icon-img" />
        </a>
        <a href="https://www.douyin.com/user/self?from_tab_name=main" target="_blank" class="nav-link platform-link">
          <img src="@/assets/tiktok.png" class="icon-img" />
        </a>
        <a href="https://space.bilibili.com/1425205072" target="_blank" class="nav-link platform-link">
          <img src="@/assets/bilibili.png" class="icon-img" />
        </a>
        <a href="https://www.xiaohongshu.com/user/profile/6487299d0000000011000c51" target="_blank" class="nav-link platform-link">
          <img src="@/assets/xiaohongshu.png" class="icon-img" />
        </a>
        <a href="https://www.douban.com/people/253295904/?_i=5057236mjxrVP7,5057247mjxrVP7" target="_blank" class="nav-link platform-link">
          <img src="@/assets/douban.png" class="icon-img" />
        </a>
        <a href="https://www.facebook.com/" target="_blank" class="nav-link platform-link">
          <img src="@/assets/facebook.png" class="icon-img" />
        </a>
        <a href="https://twitter.com/" target="_blank" class="nav-link platform-link">
          <img src="@/assets/twitter.png" class="icon-img" />
        </a>
        <a href="https://juejin.cn/user/1963060424624064" target="_blank" class="nav-link platform-link">
          <img src="@/assets/xitujuejin.png" class="icon-img" />
        </a>
        <!-- 统一样式的镂空按钮 -->
        <a href="#" class="nav-link btn-outline">RESUME</a>
        <button class="nav-link btn-outline" @click="toggleTheme">
          {{ isDark ? '☀️ 浅色' : '🌙 深色' }}
        </button>
      </div>
      <button class="mobile-menu-btn" @click="toggleMobileMenu">☰</button>
    </div>
    <div class="mobile-menu" :class="{ active: mobileMenuOpen }">

      <a href="https://github.com/BI-shi-qiang" target="_blank" class="mobile-link">
        <img src="@/assets/github.png" class="icon-img" /> GitHub
      </a>
      <a href="https://gitee.com/bi-shiqiang" target="_blank" class="mobile-link">
        <img src="@/assets/gitee.png" class="icon-img" /> Gitee
      </a>
      <a href="https://blog.csdn.net/2401_82352441?type=blog" target="_blank" class="mobile-link">
        <img src="@/assets/csdn.png" class="icon-img" /> CSDN
      </a>
      <a href="https://www.douyin.com/user/self?from_tab_name=main" target="_blank" class="mobile-link">
        <img src="@/assets/tiktok.png" class="icon-img" /> 抖音
      </a>
      <a href="https://space.bilibili.com/1425205072" target="_blank" class="mobile-link">
        <img src="@/assets/bilibili.png" class="icon-img" /> BiliBili
      </a>
      <a href="https://www.xiaohongshu.com/user/profile/6487299d0000000011000c51" target="_blank" class="mobile-link">
        <img src="@/assets/xiaohongshu.png" class="icon-img" /> 小红书
      </a>
      <a href="https://www.douban.com/people/253295904/?_i=5057236mjxrVP7,5057247mjxrVP7" target="_blank" class="mobile-link">
        <img src="@/assets/douban.png" class="icon-img" /> 豆瓣
      </a>
      <a href="https://www.facebook.com/" target="_blank" class="mobile-link">
        <img src="@/assets/facebook.png" class="icon-img" /> Facebook
      </a>
      <a href="https://twitter.com/" target="_blank" class="mobile-link">
        <img src="@/assets/twitter.png" class="icon-img" /> Twitter
      </a>
      <a href="https://juejin.cn/user/1963060424624064" target="_blank" class="mobile-link">
        <img src="@/assets/xitujuejin.png" class="icon-img" /> 掘金
      </a>
    
      <a href="#" class="mobile-link mobile-btn btn-outline">RESUME</a>
      <!-- 把 button 改成 a 标签，完全统一 -->
      <a href="javascript:;" class="mobile-link mobile-btn btn-outline" @click.prevent="toggleTheme">
        {{ isDark ? '☀️ 切换浅色' : '🌙 切换深色' }}
      </a>
    </div>
  </nav>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const isDark = ref(true)
const mobileMenuOpen = ref(false)

// 定义 emit：主题切换 + 翻页锁定
const emit = defineEmits(['theme-change', 'page-lock'])

const toggleTheme = () => {
  isDark.value = !isDark.value
  emit('theme-change', isDark.value)
}

// 菜单开关 → 发送锁定状态
const toggleMobileMenu = () => {
  mobileMenuOpen.value = !mobileMenuOpen.value
  emit('page-lock', mobileMenuOpen.value) // 关键代码
}

// 关闭菜单
const closeMobileMenu = () => {
  if (mobileMenuOpen.value) {
    mobileMenuOpen.value = false
    emit('page-lock', false)
  }
}

onMounted(() => {
  document.addEventListener('click', (e) => {
    if (!e.target.closest('.navbar')) closeMobileMenu()
  })
})

onUnmounted(() => {
  closeMobileMenu()
})
</script>

<style scoped>
/* 导航栏 */
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 9999;
  padding: 1rem 0;
  transition: all 0.3s ease;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

:global(.theme-light) .navbar {
  background: rgba(255, 255, 255, 0.9);
  border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}

/* 容器 */
.nav-container {
  max-width: 1920px;
  margin: 0 auto;
  padding: 0 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* LOGO */
.nav-brand {
  font-size: 1.5rem;
  font-weight: 700;
  text-decoration: none;
  color: #fff;
}
:global(.theme-light) .nav-brand {
  color: #000;
}

/* 导航菜单 */
.nav-menu {
  display: flex;
  align-items: center;
  gap: 1.8rem;
  flex-wrap: wrap;
}

/* 链接样式 */
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
}
.nav-link:hover {
  transform: scale(1.2);
}

:global(.theme-light) .nav-link {
  color: #000;
}

/* 图片图标 */
.icon-img {
  width: 30px;
  height: 30px;
  object-fit: contain;
}

/* 镂空按钮 */
.btn-outline {
  padding: 0.5rem 1.2rem;
  border-radius: 0px;
  border: 1px solid rgba(255, 255, 255, 0.3);
  background: transparent;
  color: #fff !important;
  cursor: pointer;
  transition: all 0.2s ease;
  text-align: center;
  display: inline-flex;
  align-items: center;
  justify-content: center;
}
:global(.theme-light) .btn-outline {
  border-color: rgba(0, 0, 0, 0.2);
  color: #000 !important;
}

.btn-outline:hover {
  opacity: 0.8;
  background: rgba(255, 255, 255, 0.1);
}
:global(.theme-light) .btn-outline:hover {
  background: rgba(0, 0, 0, 0.05);
}

/* 移动端按钮 */
.mobile-menu-btn {
  display: none;
  background: transparent;
  border: none;
  font-size: 1.8rem;
  color: #fff;
}
:global(.theme-light) .mobile-menu-btn {
  color: #000;
}

/* 移动端菜单 */
.mobile-menu {
  display: none;
  position: absolute;
  top: 100%;
  left: 0;
  width: 100%;
  background: rgba(0,0,0,0.95);
  max-height: 0;
  overflow: hidden;
  transition: all 0.3s ease;
}
:global(.theme-light) .mobile-menu {
  background: rgba(255,255,255,0.95);
}
.mobile-menu.active {
  max-height: 100vh;
  padding: 1rem 2rem;
}

.mobile-link {
  display: flex;
  align-items: center;
  gap: 0.4rem;
  padding: 0.8rem 0;
  font-size: 1rem;
  color: #fff;
  text-decoration: none;
  border-bottom: 1px solid rgba(255,255,255,0.1);
}
:global(.theme-light) .mobile-link {
  color: #000;
  border-color: rgba(0,0,0,0.1);
}

.mobile-link img {
  width: 18px;
  height: 18px;
}

/* 移动端按钮强制统一大小 */
.mobile-btn {
  width: 100%;
  margin: 0.5rem 0;
  padding: 0.8rem 0 !important;
  border-radius: 6px;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  box-sizing: border-box;
}

@media (max-width: 992px) {
  .nav-menu { display: none; }
  .mobile-menu-btn { display: block; }
  .mobile-menu { display: block; }
}
</style>