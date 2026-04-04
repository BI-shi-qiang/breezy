<template>
  <div class="home-welcome" :class="{ dark: isDark }">
    <div class="welcome-content">
      <h1 class="glitch-text">WELCOME</h1>
      <p class="word-row">
        <span>BREEZY</span>
        <span>CREATE</span>
        <span>EXPLORE</span>
      </p>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  isDark: {
    type: Boolean,
    default: false
  }
})
</script>

<style scoped>
/* 外层容器 */
.home-welcome {
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  box-sizing: border-box;
}

/* -------------------------- */
/*  LIGHT MODE 木板立体风格    */
/* -------------------------- */
.welcome-content {
  position: relative;
  width: 100%;
  max-width: 1000px;
  padding: 100px 60px;
  border-radius: 12px;

  /* 木板背景 + 立体浮雕 */
  background: #e2d5c0;
  background-image: url("@/assets/wood.webp");
  background-size: cover;
  background-position: center;
  box-shadow:
    0 12px 25px rgba(0,0,0,0.3),
    0 0 0 1px rgba(0,0,0,0.1),
    inset 0 2px 4px rgba(255,255,255,0.4),
    inset 0 -2px 4px rgba(0,0,0,0.2);
  border: 1px solid rgba(0,0,0,0.15);
  overflow: hidden;
  transition: all 0.6s ease;
}

/* 文字：雕刻质感 */
.glitch-text {
  font-size: 72px;
  font-weight: 900;
  text-align: center;
  margin: 0 0 20px 0;
  color: #3d2b1f;
  text-shadow:
    0 2px 2px rgba(255,255,255,0.5),
    0 -1px 1px rgba(0,0,0,0.3);
  letter-spacing: 6px;
}

.word-row {
  display: flex;
  justify-content: center;
  gap: 30px;
  flex-wrap: wrap;
}

.word-row span {
  font-size: 20px;
  font-weight: bold;
  color: #5a4434;
  text-shadow: 0 1px 1px rgba(255,255,255,0.4);
  letter-spacing: 2px;
}

/* -------------------------- */
/*  DARK MODE 全新科技风格     */
/* -------------------------- */
.dark .welcome-content {
  background: #0d111f;
  background-image: none;
  border: 2px solid #1a2342;
  box-shadow:
    0 0 40px #00aaff11,
    0 0 80px #00aaff08,
    inset 0 0 30px #00aaff05;
  border-radius: 16px;
  backdrop-filter: blur(10px);
}

.dark .glitch-text {
  color: #ffffff;
  text-shadow: 0 0 15px #00aaff, 0 0 30px #00aaff88;
  position: relative;
  animation: scan 3s linear infinite;
}

/* 科技扫描线动画 */
@keyframes scan {
  0% {
    filter: brightness(1);
  }
  50% {
    filter: brightness(1.2);
  }
  100% {
    filter: brightness(1);
  }
}

.dark .word-row span {
  color: #cbd5e1;
  text-shadow: 0 0 8px #00aaff55;
}

/* 响应式 */
@media (max-width: 768px) {
  .glitch-text {
    font-size: 42px;
  }
  .word-row {
    gap: 16px;
  }
  .word-row span {
    font-size: 16px;
  }
}
</style>