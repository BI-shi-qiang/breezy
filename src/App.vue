<template>
  <div
    class="viewport"
    :class="{ 'theme-dark': isDark, 'theme-light': !isDark }"
  >
    <WebBack v-if="isDark" />
    <!-- 直接使用封装好的导航栏组件 -->
    <TabVue @theme-change="onThemeChange" @page-lock="handlePageLock" :isDark="isDark" />
    <div class="wrapper" :style="wrapperStyle">
      <div class="page">
        <div class="page-content">
          <HomeWelcome :isDark="isDark" />
        </div>
      </div>
      <div class="page">
        <div class="page-content">
          <HangingTagCard :isDark="isDark" />
        </div>
      </div>
      <div class="page">
        <div class="page-content">
          <MagicCube :isDark="isDark" />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from "vue";
import WebBack from "./components/WebBack/index.vue";
import TabVue from "./components/TabVue/index.vue";
import MagicCube from "./components/MagicCube/index.vue";
import HangingTagCard from "./components/HangingTagCard/index.vue";
import HomeWelcome from "./components/HomeWelcome/index.vue";

const isDark = ref(true);
function onThemeChange(val) {
  isDark.value = val;
}

let startY = 0;
const current = ref(0);
const total = 3;
const animating = ref(false);
const pageLocked = ref(false);

const wrapperStyle = computed(() => ({
  transform: `translateY(-${current.value * 100}vh)`,
  transition: animating.value ? "transform 0.7s ease" : "none",
}));

const handlePageLock = (val) => {
  pageLocked.value = val;
};

const change = (dir) => {
  if (animating.value) return;
  if (pageLocked.value) return;
  if (dir === "up" && current.value < total - 1) {
    current.value++;
    animating.value = true;
  }
  if (dir === "down" && current.value > 0) {
    current.value--;
    animating.value = true;
  }
};

const wheel = (e) => {
  e.preventDefault();
  change(e.deltaY > 0 ? "up" : "down");
};
const touchStart = (e) => (startY = e.touches[0].clientY);
const touchMove = (e) => e.preventDefault();
const touchEnd = (e) => {
  const diff = startY - e.changedTouches[0].clientY;
  if (Math.abs(diff) > 50) change(diff > 0 ? "up" : "down");
};
const end = () => (animating.value = false);

onMounted(() => {
  const el = document.querySelector(".wrapper");
  el.addEventListener("transitionend", end);
  window.addEventListener("wheel", wheel, { passive: false });
  window.addEventListener("touchstart", touchStart, { passive: false });
  window.addEventListener("touchmove", touchMove, { passive: false });
  window.addEventListener("touchend", touchEnd, { passive: false });
});
onUnmounted(() => {
  const el = document.querySelector(".wrapper");
  el.removeEventListener("transitionend", end);
  window.removeEventListener("wheel", wheel);
  window.removeEventListener("touchstart", touchStart);
  window.removeEventListener("touchmove", touchMove);
  window.removeEventListener("touchend", touchEnd);
});
</script>

<style scoped>
.viewport {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  transition: background-color 0.3s ease;
}
.theme-dark {
  background: #000;
}
.theme-light {
  background: #faf7f0;
}

.wrapper {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  z-index: 5;
}
.page {
  width: 100%;
  height: 100%;
  flex-shrink: 0;
}
.page-content {
  width: 100vw;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  color: #faf7f0;
}
.theme-light .page-content {
  color: #000;
  background: #faf7f0;
}
.theme-dark .page-content {
  background: #000;
  color: #faf7f0;
}

/* 卡片 */
.card-list {
  display: flex;
  flex-wrap: wrap;
  gap: 30px;
  justify-content: center;
  padding: 40px 0;
}
</style>
