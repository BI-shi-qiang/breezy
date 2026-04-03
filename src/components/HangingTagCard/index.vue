<template>
  <div class="page-container">
    <!-- 吊牌卡片主体 -->
    <section class="hanging-tag-card" :class="{ 'dark-theme': isDark }">
      <!-- 顶部悬挂绳 -->
      <div class="hanger-rod"></div>

      <div class="card-content">
        <!-- 左侧：个人介绍 -->
        <div class="intro-section">
          <p class="badge">FRONTEND DEVELOPER</p>
          <h1 class="title">
            <span class="intro-text">introduce</span>
            <span class="about-me">ABOUT ME</span>
          </h1>
          <p class="description">
            Hi There, I'm BiShiqiang and I am a Front-End Web Developer based in China. It's been 1 years, I'm a student deeply passionate about front-end development. I’ve built several practice websites to sharpen my skills and enjoy turning ideas into clean, interactive experiences. Outside of coding, I love sports—especially playing table tennis, running, and cycling.
            Front-end development is where my passion truly lies, and I’m committed to growing and improving every day.
          </p>
          <a href="https://github.com/BI-shi-qiang" target="_blank" class="github-link">
            <span class="github-icon">⚫</span> https://github.com/BI-shi-qiang
          </a>
        </div>

        <!-- 右侧：头像 & 信息 -->
        <div class="sidebar-section">
          <div class="avatar-frame">
            <img 
              src="@/assets/card1.jpg" 
              alt="个人照片" 
              class="avatar-img"
            />
          </div>
          
          <div class="info-row">
            <h2 class="name">BI SHIQIANG</h2>
            <a href="/breezy/resume.pdf" target="_blank" class="resume-btn">
              <span class="download-icon">⬇</span> RESUME DOWNLOAD
            </a>
          </div>

          <div class="skills-section">
            <h3 class="skills-title">SKILLS</h3>
            <ul class="skills-list">
              <!-- <li>React Three Fiber</li> -->
              <li>Three.js</li>
              <li>Blender</li>
              <!-- <li>React.js</li> -->
              <!-- <li>React Native</li> -->
              <li>Vue.js</li>
              <li>CSS</li>
              <li>HTML</li>
              <li>Javascript</li>
              <li>Typescript</li>
            </ul>
          </div>
          
          <button class="exit-btn">EXIT</button>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref } from 'vue';

// 如果你之前的导航栏有深色模式状态，这里可以接收 props
const props = defineProps({
  isDark: {
    type: Boolean,
    default: false
  }
});
</script>

<style scoped>
/* 全局容器，用于居中展示 */
.page-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #2c2c2c; /* 深灰色背景衬托吊牌 */
  padding: 2rem;
  box-sizing: border-box;
}

/* 核心：吊牌卡片样式 */
.hanging-tag-card {
  --paper-bg: #f9f2e6; /* 复古纸张底色 */
  --shadow-color: rgba(0, 0, 0, 0.4);
  position: relative;
  width: 100%;
  max-width: 1200px;
  min-height: 600px;
  background: var(--paper-bg);
  
  /* 1. 非高清/纸张质感 */
  filter: blur(0.2px) grayscale(2%); /* 轻微模糊和灰度，模拟纸质噪点 */
  opacity: 0.95;
  
  /* 2. 吊牌悬挂效果 */
  transform: rotate(-1deg); /* 整体微旋转，更像挂上去的 */
  box-shadow: 
    0 5px 15px var(--shadow-color), /* 主投影 */
    0 0 30px rgba(0,0,0,0.1) inset; /* 内阴影，增加纸张厚度感 */
  
  /* 圆角 */
  border-radius: 4px;
}

/* 深色模式适配 */
.dark-theme .hanging-tag-card {
  --paper-bg: #2d2a24;
  --shadow-color: rgba(0, 0, 0, 0.6);
  color: #e0e0e0;
}

/* 顶部悬挂杆 */
.hanger-rod {
  position: absolute;
  top: -15px;
  left: 50%;
  transform: translateX(-50%);
  width: 60px;
  height: 15px;
  background: #555;
  border-radius: 4px 4px 0 0;
  z-index: 1;
  box-shadow: 0 -2px 5px rgba(0,0,0,0.3);
}

.hanger-rod::after {
  content: '';
  position: absolute;
  top: -8px;
  left: 50%;
  transform: translateX(-50%);
  width: 20px;
  height: 8px;
  background: #333;
  border-radius: 2px;
}

/* 卡片内容布局 */
.card-content {
  display: flex;
  padding: 40px 30px;
  gap: 40px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

/* 左侧介绍区 */
.intro-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.badge {
  font-size: 14px;
  font-weight: bold;
  letter-spacing: 1px;
  margin: 0;
}

.title {
  font-size: 48px;
  font-weight: 900;
  line-height: 1.1;
  margin: 0;
}

.intro-text {
  font-family: 'Brush Script MT', cursive; /* 保留手写体风格，系统无 */
  color: #d32f2f;
  font-weight: 500;
}

.about-me {
  display: block;
  color: #333;
}

.dark-theme .about-me {
  color: #fff;
}

.description {
  font-size: 16px;
  line-height: 1.6;
  color: #444;
  margin: 10px 0;
}

.dark-theme .description {
  color: #ccc;
}

.github-link {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  color: #000;
  text-decoration: none;
  font-weight: 500;
  margin-top: 10px;
}

.dark-theme .github-link {
  color: #fff;
}

.github-link:hover {
  text-decoration: underline;
}

/* 右侧边栏 */
.sidebar-section {
  width: 320px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

/* 头像相框 */
.avatar-frame {
  width: 280px;
  height: 380px;
  border: 8px solid #d32f2f;
  border-radius: 4px;
  overflow: hidden;
  box-shadow: 0 4px 10px rgba(0,0,0,0.2);
}

.avatar-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

/* 名字与下载 */
.name {
  font-size: 24px;
  font-weight: bold;
  letter-spacing: 2px;
  margin: 0;
  color: #333;
}

.dark-theme .name {
  color: #fff;
}

.resume-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  border: 1px solid #333;
  color: #333;
  text-decoration: none;
  font-weight: 500;
  font-size: 14px;
  transition: all 0.2s;
}

.dark-theme .resume-btn {
  border-color: #fff;
  color: #fff;
}

.resume-btn:hover {
  background: rgba(0,0,0,0.1);
}

/* 技能列表 */
.skills-title {
  font-size: 18px;
  font-weight: bold;
  margin: 0 0 10px 0;
  color: #333;
  align-self: flex-start;
}

.dark-theme .skills-title {
  color: #fff;
}

.skills-list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-wrap: wrap;
  gap: 10px 15px;
  justify-content: flex-start;
}

.skills-list li {
  font-size: 14px;
  color: #555;
  padding: 4px 0;
}

.dark-theme .skills-list li {
  color: #ddd;
}

/* 退出按钮 */
.exit-btn {
  padding: 8px 20px;
  background: #333;
  color: #fff;
  border: none;
  font-weight: bold;
  cursor: pointer;
  margin-top: 20px;
  align-self: flex-end;
}

.dark-theme .exit-btn {
  background: #fff;
  color: #333;
}

/* 响应式：移动端适配 */
@media (max-width: 900px) {
  .card-content {
    flex-direction: column;
    align-items: center;
  }
  
  .sidebar-section {
    width: 100%;
  }
  
  .title {
    font-size: 36px;
  }
}
</style>