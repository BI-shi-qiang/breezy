<h1 align="center">
  <img src="https://readme-typing-svg.herokuapp.com?font=Fira+Code&weight=700&size=40&duration=3000&pause=1000&color=00AAFF&center=true&vCenter=true&width=600&height=70&lines=%E6%AF%95%E4%B8%96%E5%BC%BA%E7%9A%84%E4%B8%AA%E4%BA%BA%E7%BD%91%E7%AB%99" alt="shiqiang's personal website" />
</h1>

<p align="center">
  <img src="https://img.shields.io/badge/Vue.js-3.5-4FC08D?style=for-the-badge&logo=vuedotjs&logoColor=white" alt="Vue.js" />
  <img src="https://img.shields.io/badge/Vite-8.0-646CFF?style=for-the-badge&logo=vite&logoColor=white" alt="Vite" />
  <img src="https://img.shields.io/badge/Three.js-0.183-000000?style=for-the-badge&logo=threedotjs&logoColor=white" alt="Three.js" />
  <img src="https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black" alt="JavaScript" />
  <img src="https://img.shields.io/badge/Socket.IO-4.8-010101?style=for-the-badge&logo=socketdotio&logoColor=white" alt="Socket.IO" />
  <img src="https://img.shields.io/badge/GitHub_Pages-Deployed-222222?style=for-the-badge&logo=githubpages&logoColor=white" alt="GitHub Pages" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/status-active-success?style=flat-square" alt="Status" />
  <img src="https://img.shields.io/badge/license-private-red?style=flat-square" alt="License" />
  <img src="https://img.shields.io/github/last-commit/BI-shi-qiang/breezy?style=flat-square&color=blue" alt="Last Commit" />
</p>

---

## 📖 项目简介

**Breezy** 是我的个人品牌网站，集作品展示、个人介绍、3D 交互体验与 AI 智能助手于一体。网站采用全屏滚动式设计，支持明暗双主题切换，力求在视觉呈现与交互体验上做到与众不同。

> Hi, I'm **Bi Shiqiang** — a front-end web developer based in China. As a student with a deep passion for coding, I enjoy turning ideas into clean, interactive websites. Outside of development, I love sports like table tennis, running, and cycling. I am committed to growing and improving in this field every day.

---

## ✨ 核心功能

| 模块 | 说明 |
|------|------|
| 🏠 **首页欢迎屏** | 故障风格文字特效 + 木板 / 科技感双主题背景 |
| 🏷️ **个人名片** | 悬挂吊牌式卡片，展示个人信息、技能标签与简历下载 |
| 🧊 **3D 魔方** | 基于 Three.js 的交互式 3D 魔方，支持拖拽旋转 |
| 🤖 **AI 助手** | 智能聊天助手，集成 AI 对话能力 |
| 🌓 **主题切换** | 明暗双主题一键切换，暗色模式带霓虹发光边框效果 |
| 📜 **全屏滚动** | 鼠标滚轮 / 触摸滑动驱动整页切换，流畅过渡动画 |

---

## 🛠️ 技术栈

### 前端框架

![Vue.js](https://img.shields.io/badge/Vue.js-3.5-4FC08D?style=flat&logo=vuedotjs&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-8.0-646CFF?style=flat&logo=vite&logoColor=white)

### 3D 渲染

![Three.js](https://img.shields.io/badge/Three.js-0.183-000000?style=flat&logo=threedotjs&logoColor=white)
![Blender](https://img.shields.io/badge/Blender-建模-F5792A?style=flat&logo=blender&logoColor=white)

### 实时通信

![Socket.IO](https://img.shields.io/badge/Socket.IO-4.8-010101?style=flat&logo=socketdotio&logoColor=white)

### 工具 & 部署

![GitHub Pages](https://img.shields.io/badge/GitHub_Pages-部署-222222?style=flat&logo=githubpages&logoColor=white)
![gh-pages](https://img.shields.io/badge/gh--pages-6.3-24292e?style=flat&logo=npm&logoColor=white)

### 语言

![JavaScript](https://img.shields.io/badge/JavaScript-ES6+-F7DF1E?style=flat&logo=javascript&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-类型定义-3178C6?style=flat&logo=typescript&logoColor=white)
![HTML5](https://img.shields.io/badge/HTML5-语义化-E34F26?style=flat&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-动画/响应式-1572B6?style=flat&logo=css3&logoColor=white)

---

## 🚀 快速开始

### 环境要求

- **Node.js** >= 18
- **npm** >= 9

### 安装与运行

```bash
# 克隆仓库
git clone https://github.com/BI-shi-qiang/breezy.git

# 进入项目目录
cd breezy

# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览构建结果
npm run preview

# 部署到 GitHub Pages
npm run deploy
```

开发服务器默认运行在 `http://localhost:5173`。

---

## 📁 项目结构

```
breezy/
├── public/                     # 静态资源
│   └── favicon.svg
├── src/
│   ├── assets/                 # 图片 / 字体等资源
│   ├── components/
│   │   ├── HomeWelcome/        # 首页欢迎屏
│   │   ├── HangingTagCard/     # 个人名片卡片
│   │   ├── MagicCube/          # 3D 魔方交互
│   │   ├── AiAssistant/        # AI 聊天助手
│   │   ├── WebBack/            # 动态背景
│   │   ├── TabVue/             # 导航栏 & 主题切换
│   │   └── ScreenFull/         # 全屏控制
│   ├── App.vue                 # 根组件（页面路由 & 滚动逻辑）
│   ├── main.js                 # 应用入口
│   └── style.css               # 全局样式
├── index.html                  # HTML 入口
├── vite.config.js              # Vite 配置
├── package.json                # 项目依赖
└── README.md
```

---

## 🙋‍♂️ 关于我

| 项目 | 内容 |
|------|------|
| **姓名** | 毕世强（Bi Shiqiang） |
| **职业** | 前端开发工程师 / 学生 |
| **所在地** | 中国 |
| **GitHub** | [BI-shi-qiang](https://github.com/BI-shi-qiang) |
| **技能** | HTML · CSS · JavaScript · Vue.js · Three.js · TypeScript · Blender |
| **爱好** | 🏓 乒乓球 · 🏃 跑步 · 🚴 骑行 · 💻 写代码 |

---

## 📄 许可证

本项目为个人私有项目，仅供展示与学习参考。

---

<p align="center">
  <sub>Made with ❤️ by Bi Shiqiang · &copy; 2025</sub>
</p>
