<template>
  <div class="page-container">
    <section class="hanging-tag-card" :class="{ 'dark-theme': isDark }">
      <div class="hanger-rod"></div>

      <div class="card-content">
        <div class="intro-section">
          <p class="badge">FRONTEND DEVELOPER</p>
          <h1 class="title">
            <span class="intro-text">introduce</span>
            <span class="about-me">ABOUT ME</span>
          </h1>
          <p class="description">
            Hi, I'm Bi Shiqiang, a front-end web developer based in China. As a student with a deep passion for coding, I enjoy turning ideas into clean, interactive websites. Outside of development, I love sports like table tennis, running, and cycling. I am committed to growing and improving in this field every day.
          </p>
          <a href="https://github.com/BI-shi-qiang" target="_blank" class="github-link">
            <img src="@/assets/github.png" class="icon-img" /> https://github.com/BI-shi-qiang
          </a>
        </div>

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
              <li>HTML</li>
              <li>CSS</li>
              <li>Javascript</li>
              <li>Vue.js</li>
              <li>Three.js</li>
              <li>Blender</li>
              <li>Typescript</li>
            </ul>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
const props = defineProps({
  isDark: {
    type: Boolean,
    default: false
  }
});
</script>

<style scoped>
.page-container {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem;
  box-sizing: border-box;
}

/* 基础吊牌卡片（亮色模式） */
.hanging-tag-card {
  --paper-bg: #f9f2e6;
  --shadow-color: rgba(0, 0, 0, 0.4);
  --glow-color: transparent;
  position: relative;
  width: 100%;
  max-width: 1200px;
  min-height: 600px;
  background: var(--paper-bg);
  filter: blur(0.2px) grayscale(2%);
  opacity: 0.95;
  transform: rotate(-1deg);
  box-shadow: 
    0 5px 15px var(--shadow-color),
    0 0 0 2px var(--glow-color),
    0 0 30px rgba(0,0,0,0.1) inset;
  border-radius: 4px;
  transition: all 0.5s ease;
}

/* 🌟 深色模式 → 科技感 + 发光边框 */
.hanging-tag-card.dark-theme {
  --paper-bg: #111827;
  --shadow-color: rgba(0,0,0,0.8);
  --glow-color: #00aaff;
  transform: rotate(0deg);
  background: #0f172a;
  border: 2px solid #1e293b;
  box-shadow: 
    0 0 25px #00aaff66,
    0 0 60px #00aaff33,
    0 0 100px #00aaff11,
    0 0 0 2px var(--glow-color);
  color: #e0e0e0;
}

/* 🌟 深色模式 → 隐藏吊绳 */
.dark-theme .hanger-rod {
  opacity: 0;
  visibility: hidden;
}

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
  transition: all 0.4s ease;
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

.card-content {
  display: flex;
  padding: 40px 30px;
  gap: 40px;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

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
  color: #d32f2f;
}
.dark-theme .badge {
  color: #00aaff;
}

.title {
  font-size: 48px;
  font-weight: 900;
  line-height: 1.1;
  margin: 0;
}

.intro-text {
  font-family: 'Brush Script MT', cursive;
  color: #d32f2f;
  font-weight: 500;
}
.dark-theme .intro-text {
  color: #00aaff;
}

.about-me {
  display: block;
  color: #333;
}
.dark-theme .about-me {
  color: #ffffff;
}

.description {
  font-size: 16px;
  line-height: 1.6;
  color: #444;
  margin: 10px 0;
}
.dark-theme .description {
  color: #cbd5e1;
}
.icon-img {
  width: 25px;
  height: 25px;
  object-fit: contain;
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
  color: #00aaff;
}

.github-link:hover {
  text-decoration: underline;
}

.sidebar-section {
  width: 320px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.avatar-frame {
  width: 280px;
  height: 380px;
  border: 8px solid #d32f2f;
  border-radius: 4px;
  overflow: hidden;
  box-shadow: 0 4px 10px rgba(0,0,0,0.2);
  transition: all 0.4s ease;
}
.dark-theme .avatar-frame {
  border-color: #00aaff;
  box-shadow: 0 0 20px #00aaff55;
}

.avatar-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.name {
  font-size: 24px;
  font-weight: bold;
  letter-spacing: 2px;
  margin: 0;
  color: #333;
}
.dark-theme .name {
  color: #ffffff;
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
  border-color: #00aaff;
  color: #00aaff;
  background: #1e293b;
}

.resume-btn:hover {
  background: rgba(0,0,0,0.1);
}
.dark-theme .resume-btn:hover {
  background: #1e40af22;
}

.skills-title {
  font-size: 18px;
  font-weight: bold;
  margin: 0 0 10px 0;
  color: #333;
  align-self: flex-start;
}
.dark-theme .skills-title {
  color: #00aaff;
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
  color: #cbd5e1;
}

@media (max-width: 900px) {
  .card-content {
    flex-direction: column;
    align-items: center;
  }
  .avatar-frame {
    width: 300px;
    height: 200px;
  }
  .sidebar-section {
    width: 100%;
  }
  
  .title {
    font-size: 20px;
  }
  .description {
    margin-top: 0;
  }
}
</style>