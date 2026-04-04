<template>
  <div
    class="navbar-wrapper"
    :class="[
      { hidden: navHidden },
      isDark ? 'theme-dark' : 'theme-light'
    ]"
    ref="wrapperRef"
  >
    <audio ref="audio" autoplay loop>
      <source src="/breezy/bg-music.ogg" type="audio/ogg" />
    </audio>

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
          <a href="https://www.douyin.com/user/self?from_tab_name=main" target="_blank" class="nav-link platform-link">
            <img src="@/assets/tiktok.png" class="icon-img" />
          </a>
          <a href="https://blog.csdn.net/2401_82352441?ops_request_misc=%7B%22request%5Fid%22%3A%221faa12bc8c8ea0335bdcf3309ea5b353%22%2C%22scm%22%3A%2220140713.130064515..%22%7D&request_id=1faa12bc8c8ea0335bdcf3309ea5b353&biz_id=206&utm_medium=distribute.pc_search_result.none-task-user-null-1-169420311-null-null.nonecase&utm_term=%E7%AA%9D%E5%AD%90%E9%9D%A2&spm=1018.2226.3001.4351" target="_blank" class="nav-link platform-link">
            <img src="@/assets/csdn.png" class="icon-img" />
          </a>
          <a href="https://www.xiaohongshu.com/user/profile/6487299d0000000011000c51" target="_blank" class="nav-link platform-link">
            <img src="@/assets/xiaohongshu.png" class="icon-img" />
          </a>
          <a href="https://www.douban.com/people/253295904/?_i=5057236mjxrVP7,5057236mjxrVP7" target="_blank" class="nav-link platform-link">
            <img src="@/assets/douban.png" class="icon-img" />
          </a>
          <a href="https://space.bilibili.com/1425205072" target="_blank" class="nav-link platform-link">
            <img src="@/assets/bilibili.png" class="icon-img" />
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
          
          <button class="nav-link icon-btn" @click="toggleMusic">
            <img
              v-if="isMusicPlaying"
              src="@/assets/music-on.png"
              class="icon-img"
            />
            <img
              v-else
              src="@/assets/music-off.png"
              class="icon-img"
            />
          </button>

          <button class="nav-link icon-btn" @click="toggleTheme">
            <img
              v-if="isDark"
              src="@/assets/sun.png"
              class="icon-img"
            />
            <img
              v-else
              src="@/assets/moon.png"
              class="icon-img"
            />
          </button>
        </div>
        <button class="mobile-menu-btn" @click.stop="toggleMobileMenu">
          <img 
            v-if="mobileMenuOpen" 
            src="@/assets/close.png" 
            class="icon-img" 
          />
          <img 
            v-else 
            src="@/assets/menu.png" 
            class="icon-img" 
          />
        </button>
      </div>

      <div class="mobile-menu" :class="{ active: mobileMenuOpen }">
        <a href="https://github.com/BI-shi-qiang" target="_blank" class="mobile-link">
          <img src="@/assets/github.png" class="icon-img" /> GitHub
        </a>
        <a href="https://gitee.com/bi-shiqiang" target="_blank" class="mobile-link">
          <img src="@/assets/gitee.png" class="icon-img" /> Gitee
        </a>
        <a href="https://www.douyin.com/user/self?from_tab_name=main" target="_blank" class="mobile-link">
          <img src="@/assets/tiktok.png" class="icon-img" /> 抖音
        </a>
        <a href="https://blog.csdn.net/2401_82352441?ops_request_misc=%7B%22request%5Fid%22%3A%221faa12bc8c8ea0335bdcf3309ea5b353%22%2C%22scm%22%3A%2220140713.130064515..%22%7D&request_id=1faa12bc8c8ea0335bdcf3309ea5b353&biz_id=206&utm_medium=distribute.pc_search_result.none-task-user-null-1-169420311-null-null.nonecase&utm_term=%E7%AA%9D%E5%AD%90%E9%9D%A2&spm=1018.2226.3001.4351" target="_blank" class="mobile-link">
          <img src="@/assets/csdn.png" class="icon-img" /> CSDN
        </a>
        <a href="https://www.xiaohongshu.com/user/profile/6487299d0000000011000c51" target="_blank" class="mobile-link">
          <img src="@/assets/xiaohongshu.png" class="icon-img" /> 小红书
        </a>
        <a href="https://www.douban.com/people/253295904/?_i=5057236mjxrVP7,5057236mjxrVP7" target="_blank" class="mobile-link">
          <img src="@/assets/douban.png" class="icon-img" /> 豆瓣
        </a>
        <a href="https://space.bilibili.com/1425205072" target="_blank" class="mobile-link">
          <img src="@/assets/bilibili.png" class="icon-img" /> BiliBili
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
        
        <a href="javascript:;" class="mobile-link mobile-btn btn-outline" @click.prevent="toggleMusic">
          <img 
            v-if="isMusicPlaying" 
            src="@/assets/music-on.png" 
            class="icon-img" 
          />
          <img 
            v-else 
            src="@/assets/music-off.png" 
            class="icon-img" 
          />
        </a>

        <a href="javascript:;" class="mobile-link mobile-btn btn-outline" @click.prevent="toggleTheme">
          <img 
            v-if="isDark" 
            src="@/assets/sun.png" 
            class="icon-img" 
          />
          <img 
            v-else 
            src="@/assets/moon.png" 
            class="icon-img" 
          />
        </a>
      </div>
    </nav>

    <div class="rope-container" @click="toggleNavbar" :class="{ swinging: isSwinging }">
      <div class="rope"></div>
      <div class="rope-puller"></div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, defineProps, defineEmits } from 'vue'

// 接收父组件主题
const props = defineProps({
  isDark: {
    type: Boolean,
    default: true
  }
})

const isDark = computed(() => props.isDark)

const mobileMenuOpen = ref(false)
const navHidden = ref(true)
const isSwinging = ref(false)
const wrapperRef = ref(null)
const firstTab = ref(0)

const isMusicPlaying = ref(false)
const audio = ref()

const plays = () => {
  if (audio.value) {
    audio.value.play()
  }
}

const toggleMusic = () => {
  if (!audio.value) return
  if (isMusicPlaying.value) {
    audio.value.pause()
    isMusicPlaying.value = false
  } else {
    audio.value.play().catch(err => {
      console.log('播放被限制', err)
    })
    isMusicPlaying.value = true
  }
}

const emit = defineEmits(['theme-change', 'page-lock'])

const toggleTheme = () => {
  emit('theme-change', !props.isDark)
}

const toggleMobileMenu = () => {
  mobileMenuOpen.value = !mobileMenuOpen.value
  emit('page-lock', mobileMenuOpen.value)
}

const closeMobileMenu = () => {
  if (mobileMenuOpen.value) {
    mobileMenuOpen.value = false
    emit('page-lock', false)
  }
}

const toggleNavbar = () => {
  if (!isMusicPlaying.value && firstTab.value === 0) {
    isMusicPlaying.value = true
    plays()
    firstTab.value = 1
  }
  navHidden.value = !navHidden.value
  isSwinging.value = true
  setTimeout(() => {
    isSwinging.value = false
  }, 800)
}

onMounted(() => {
  document.addEventListener('click', (e) => {
    if (!e.target.closest('.navbar')) closeMobileMenu()
  })
})

onUnmounted(() => {
  closeMobileMenu()
  if (audio.value) {
    audio.value.pause()
  }
})
</script>

<style scoped>
.navbar-wrapper {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 9999;
  transition: transform 0.5s cubic-bezier(0.25, 0.8, 0.25, 1);
}
.navbar-wrapper.hidden {
  transform: translateY(-100%);
}

/* 深色模式导航栏 */
.navbar {
  padding: 1rem 0;
  transition: all 0.3s ease;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

/* 浅色模式导航栏 */
.theme-light .navbar {
  background: rgba(250, 247, 240, 0.9);
  border-bottom: 1px solid rgba(0, 0, 0, 0.08);
}

.nav-container {
  max-width: 1920px;
  margin: 0 auto;
  padding: 0 2rem;
  margin-right: 2rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.nav-brand {
  font-size: 1.5rem;
  font-weight: 700;
  text-decoration: none;
  color: #ffffff;
}
.theme-light .nav-brand {
  color: #222;
}

.nav-menu {
  display: flex;
  align-items: center;
  gap: 1.8rem;
  flex-wrap: wrap;
}

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
.theme-light .btn-outline {
  border-color: rgba(0, 0, 0, 0.2);
  color: #222 !important;
}

.btn-outline:hover {
  opacity: 0.8;
  background: rgba(255, 255, 255, 0.1);
}
.theme-light .btn-outline:hover {
  background: rgba(0, 0, 0, 0.05);
}

.mobile-menu-btn {
  padding: 0 !important;
  border: none !important;
  background: transparent !important;
  display: none;
  transition: transform 0.3s ease;
}
.mobile-menu-btn img {
  width: 30px;
  height: 30px;
  object-fit: contain;
}
.mobile-menu-btn.active {
  transform: rotate(90deg);
}

@media (max-width: 992px) {
  .mobile-menu-btn {
    display: block;
  }
}

.theme-light .mobile-menu-btn {
  color: #000;
}

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
.theme-light .mobile-menu {
  background: rgba(250, 247, 240, 0.98);
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
.theme-light .mobile-link {
  color: #222;
  border-color: rgba(0,0,0,0.08);
}

.mobile-link img {
  width: 18px;
  height: 18px;
}

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

.rope-container {
  position: fixed;
  top: 0;
  right: 40px;
  width: 2px;
  z-index: 9998;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  align-items: center;
  transform-origin: top center;
}

.rope-container.swinging {
  animation: ropeSwing 0.8s ease-in-out;
}

@keyframes ropeSwing {
  0%   { transform: rotate(0deg); }
  20%  { transform: rotate(8deg); }
  40%  { transform: rotate(-6deg); }
  60%  { transform: rotate(3deg); }
  80%  { transform: rotate(-2deg); }
  100% { transform: rotate(0deg); }
}

.rope {
  width: 2px;
  height: 120px;
  background: rgba(255, 255, 255, 0.4);
  transition: height 0.4s ease;
}
.theme-light .rope {
  background: rgba(0, 0, 0, 0.3);
}

.rope-puller {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  background: #fff;
  box-shadow: 0 0 5px rgba(0,0,0,0.3);
  transition: transform 0.2s ease;
}
.theme-light .rope-puller {
  background: #000;
}

.rope-container:active .rope-puller {
  transform: scale(0.8);
}

@media (max-width: 992px) {
  .nav-menu { display: none; }
  .mobile-menu-btn { display: block; }
  .mobile-menu { display: block; }
}
</style>