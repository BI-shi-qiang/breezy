<template>
  <div class="ai-float-assistant">
    <!-- 3D 小人（取代原来的 💬 悬浮球） -->
    <div class="character-ball" :class="{ isDark: isDark }" @click="onCharacterClick">
      <canvas ref="canvas" class="character-canvas"></canvas>
      <div v-if="modelLoading" class="char-loading">加载中…</div>
    </div>

    <!-- 聊天窗口 -->
    <div class="chat-window" ref="chatWindowEl" v-show="isChatOpen" :class="{ isDark: isDark }" @click.stop>
      <div class="chat-header" :class="{ isDark: isDark }">
        <span>阿毕</span>
        <button class="close-btn" :class="{ isDark: isDark }" @click="closeChat">×</button>
      </div>

      <div class="chat-container">
        <div class="message-list" ref="messageListEl">
          <div v-for="(msg, idx) in messages" :key="idx" class="message" :class="msg.role">
            <!-- AI 消息 → 加头像 -->
            <div v-if="msg.role === 'ai'" class="avatar-box">
              <img src="@/assets/avatar.jpg" alt="AI" class="avatar" />
            </div>

            <div class="msg-body">
              <div v-if="msg.content.trim()" class="msg-content">{{ msg.content }}</div>
              <!-- 引用来源 -->
              <div v-if="msg.sources && msg.sources.length" class="sources">
                <span class="sources-title">参考来源：</span>
                <span v-for="(s, i) in msg.sources" :key="i" class="source-tag">
                  {{ s.title }} · v{{ s.version }}
                </span>
              </div>
            </div>
          </div>

          <div v-if="isLoading" class="message ai loading">
            <div class="msg-content">助手正在思考中...</div>
          </div>
        </div>

        <div class="input-bar">
          <input
            v-model="input"
            @keyup.enter="sendMessage"
            placeholder="请输入问题..."
            class="input"
          />
          <button class="send-btn" :class="{ isDark: isDark }" @click="sendMessage" :disabled="isLoading">
            {{ isLoading ? "发送中..." : "发送" }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick, onMounted, onUnmounted, computed } from "vue";
import * as THREE from "three";
import { GLTFLoader } from "three/addons/loaders/GLTFLoader.js";
import { RoomEnvironment } from "three/addons/environments/RoomEnvironment.js";

const props = defineProps({
  pageLocked: {
    type: Boolean,
    default: false,
  },
  isDark: {
    type: Boolean,
    default: true,
  },
});
const emit = defineEmits(["toggle-page-lock"]);
const isDark = computed(() => props.isDark)
// RAG 公开问答接口（指向线上后端）
const RAG_API = "https://heyuan.ink/api/v1/rag/knowledge/public";

// ============ 聊天状态 ============
const input = ref("");
const messages = ref<any[]>([
  { role: "ai", content: "你好，我是阿毕小助手，可以问关于“我”与“荷源”的一切问题～" },
]);
const isLoading = ref(false);
const isChatOpen = ref(false);
const messageListEl = ref<HTMLElement | null>(null);
const chatWindowEl = ref<HTMLElement | null>(null);

// ============ 3D 小人状态 ============
const canvas = ref<HTMLCanvasElement | null>(null);
const modelLoading = ref(true);

let renderer, scene, camera, mixer, model, standAction;
let intent = null; // 当前动画意图：'stand' | 'lie' | null
let rafId = 0;
const clock = new THREE.Clock();
const ANIM_SPEED = 1.6; // 起身/躺下动画播放速度倍率（可调）

// 开/关聊天框，并通知父组件是否锁定页面滚动
const setChatOpen = (open: boolean) => {
  isChatOpen.value = open;
  emit("toggle-page-lock", open);
};

// 点击小人 → 起身 → 起身完成后再打开聊天框
const onCharacterClick = () => {
  if (isChatOpen.value || modelLoading.value || intent) return;
  playStandUp();
};

// 点击 × → 先关聊天框 → 小人躺下
const closeChat = () => {
  if (intent) return;
  setChatOpen(false);
  playLieDown();
};

// 点击聊天框以外的区域也关闭聊天框（并让小人躺下）
const onDocClick = (e: MouseEvent) => {
  if (!isChatOpen.value) return;
  if (chatWindowEl.value && chatWindowEl.value.contains(e.target as Node)) return;
  closeChat();
};

// 起身：正放动画
const playStandUp = () => {
  intent = "stand";
  if (model) model.scale.y = 1; // 复位呼吸起伏
  standAction.reset();
  standAction.time = 0;
  standAction.timeScale = ANIM_SPEED;
  standAction.setLoop(THREE.LoopOnce, 1);
  standAction.clampWhenFinished = true;
  standAction.play();
};

// 躺下：倒放动画
const playLieDown = () => {
  intent = "lie";
  if (model) model.scale.y = 1; // 复位呼吸起伏
  standAction.reset();
  standAction.time = standAction.getClip().duration;
  standAction.timeScale = -ANIM_SPEED;
  standAction.setLoop(THREE.LoopOnce, 1);
  standAction.clampWhenFinished = true;
  standAction.play();
};

// ============ three.js 场景 ============
function initScene() {
  const el = canvas.value;
  const size = el.clientWidth || 130;

  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(40, 1, 0.05, 100);

  renderer = new THREE.WebGLRenderer({ canvas: el, alpha: true, antialias: true });
  renderer.setSize(size, size);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  renderer.setClearColor(0x000000, 0);
  renderer.outputColorSpace = THREE.SRGBColorSpace;
  renderer.toneMapping = THREE.ACESFilmicToneMapping;
  renderer.toneMappingExposure = 1.0;

  // 灯光：天光 + 主光 + 轮廓光
  scene.add(new THREE.HemisphereLight(0xffffff, 0x444444, 0.9));
  const key = new THREE.DirectionalLight(0xffffff, 2.2);
  key.position.set(1.5, 2, 2);
  scene.add(key);
  const rim = new THREE.DirectionalLight(0xffffff, 0.6);
  rim.position.set(-1, 0.5, -1);
  scene.add(rim);

  // 环境贴图，给 PBR 材质提供反射
  const pmrem = new THREE.PMREMGenerator(renderer);
  scene.environment = pmrem.fromScene(new RoomEnvironment(), 0.04).texture;
  pmrem.dispose();

  const loader = new GLTFLoader();
  loader.load(
    "/models/assistant.glb",
    (gltf) => {
      model = gltf.scene;
      scene.add(model);
      frameModel(model);

      mixer = new THREE.AnimationMixer(model);
      const clip = THREE.AnimationClip.findByName(gltf.animations, "StandUp");
      standAction = mixer.clipAction(clip);

      // 初始摆到「躺下」姿势（动画第 0 帧）
      standAction.play();
      standAction.paused = true;
      standAction.time = 0;
      mixer.update(0);

      mixer.addEventListener("finished", onAnimationFinished);

      modelLoading.value = false;
      render();
    },
    undefined,
    (err) => {
      console.error("[AiAssistant] 模型加载失败:", err);
      modelLoading.value = false;
    }
  );
}

// 根据模型包围盒框景（躺下/站立两姿态长轴都约等于身高，按身高框景即可）
function frameModel(model) {
  const box = new THREE.Box3().setFromObject(model);
  const size = box.getSize(new THREE.Vector3());
  const center = box.getCenter(new THREE.Vector3());

  const height = Math.max(size.y, size.z) * 1.3;
  const fov = (camera.fov * Math.PI) / 180;
  const dist = height / 2 / Math.tan(fov / 2);

  const dir = new THREE.Vector3(0.4, 0.35, 1).normalize();
  camera.position.copy(center).addScaledVector(dir, dist);
  camera.lookAt(center);
  camera.updateProjectionMatrix();
}

// 动画播完后处理状态
function onAnimationFinished() {
  if (intent === "stand") {
    intent = null;
    setChatOpen(true); // 起身完成，打开聊天框
  } else if (intent === "lie") {
    intent = null;
  }
}

// 常驻渲染循环：一直运行，小人空闲时会自己呼吸起伏
let breatheTime = 0;

function animate() {
  rafId = requestAnimationFrame(animate);
  const delta = clock.getDelta();
  if (mixer) {
    mixer.update(delta);
    // 空闲时呼吸起伏（自己动）
    if (model && !intent) {
      breatheTime += delta;
      model.scale.y = 1 + Math.sin(breatheTime * 2.2) * 0.02;
    }
  }
  if (renderer) renderer.render(scene, camera);
}

function render() {
  if (renderer) renderer.render(scene, camera);
}

// ============ 聊天逻辑 ============
const scrollToBottom = () => {
  nextTick(() => {
    if (messageListEl.value) {
      messageListEl.value.scrollTop = messageListEl.value.scrollHeight;
    }
  });
};

// 发送消息 → 调 RAG 接口
const sendMessage = async () => {
  const text = input.value.trim();
  if (!text || isLoading.value) return;

  messages.value.push({ role: "user", content: text });
  input.value = "";
  isLoading.value = true;
  scrollToBottom();

  try {
    const res = await fetch(RAG_API, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ question: text }),
    });
    const json = await res.json();

    if (json && json.data) {
      const { answer, sources } = json.data;
      messages.value.push({
        role: "ai",
        content: answer ?? "（未获取到回答）",
        sources: sources ?? [],
      });
    } else {
      messages.value.push({
        role: "ai",
        content: json?.msg || "请求失败，请稍后再试",
      });
    }
  } catch (err) {
    messages.value.push({ role: "ai", content: "网络异常，请稍后再试" });
  } finally {
    isLoading.value = false;
    scrollToBottom();
  }
};

onMounted(() => {
  initScene();
  animate();
  document.addEventListener("click", onDocClick);
});

onUnmounted(() => {
  document.removeEventListener("click", onDocClick);
  if (rafId) cancelAnimationFrame(rafId);
  mixer?.removeEventListener("finished", onAnimationFinished);
  // 释放 GPU 资源
  if (model) {
    model.traverse((o) => {
      if (o.isMesh) {
        o.geometry?.dispose();
        const mats = Array.isArray(o.material) ? o.material : [o.material];
        mats.forEach((m) => m?.dispose());
      }
    });
  }
  renderer?.dispose();
});
</script>

<style scoped>
/* 全局容器 */
.ai-float-assistant {
  position: fixed;
  right: 20px;
  bottom: 20px;
  z-index: 9999;
}

/* 3D 小人悬浮球 */
.character-ball {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  background: radial-gradient(circle at 50% 28%, rgba(255, 255, 255, 0.55), rgba(205, 222, 240, 0.3));
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.12);
  transition: transform 0.25s ease, box-shadow 0.25s ease, background 0.3s ease;
}
.character-ball:hover {
  transform: scale(1.06);
  box-shadow: 0 6px 22px rgba(0, 0, 0, 0.2);
}
/* 深色主题下的悬浮球：暗色半透明 */
.character-ball.isDark {
  background: radial-gradient(circle at 50% 28%, rgba(40, 45, 60, 0.55), rgba(20, 24, 36, 0.3));
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.4);
}
.character-canvas {
  display: block;
  width: 100%;
  height: 100%;
}
.char-loading {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #999;
  font-size: 12px;
  letter-spacing: 1px;
  pointer-events: none;
}

/* 聊天窗口 */
.chat-window {
  position: fixed;
  right: 110px;
  bottom: 110px;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
  background: white;
  border-radius: 0;
  box-shadow: 0 8px 30px rgba(0, 0, 0, 0.15);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

/* PC 端恢复圆角大小 */
@media (min-width: 768px) {
  .chat-window {
    width: 480px;
    height: 620px;
    border-radius: 16px 0 0 0;
    left: auto;
    top: auto;
  }
}

.chat-header {
  padding: 14px 16px;
  background: #f9f2e6;
  color: #333;
  font-weight: 500;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
}
.chat-header.isDark {
  background: #333;
  color: #fff;
}
.close-btn {
  background: none;
  border: none;
  color: #333;
  font-size: 20px;
  cursor: pointer;
  line-height: 1;
}
.close-btn.isDark {
  color: #fff;
}


/* 聊天区域 */
.chat-container {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 12px;
  overflow: hidden;
}
.message-list {
  flex: 1;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 12px;
  scrollbar-width: none;
  -ms-overflow-style: none;
}
.message-list::-webkit-scrollbar {
  display: none;
}
.message {
  display: flex;
  align-items: flex-end;
  gap: 8px;
  max-width: 85%;
}
.message.ai {
  align-self: flex-start;
}
.message.user {
  align-self: flex-end;
  flex-direction: row-reverse;
}

/* 头像 */
.avatar-box {
  flex-shrink: 0;
}
.avatar {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  object-fit: cover;
}

/* 消息气泡 */
.msg-body {
  display: flex;
  flex-direction: column;
}
.msg-content {
  padding: 10px 14px;
  border-radius: 14px;
  font-size: 14px;
  line-height: 1.5;
  word-wrap: break-word;
  white-space: pre-wrap;
}
.message.user .msg-content {
  background: #05949f;
  color: white;
  border-bottom-right-radius: 2px;
}
.message.ai .msg-content {
  background: #e5e7eb;
  color: #333;
  border-bottom-left-radius: 2px;
}
.message.loading .msg-content {
  color: #666;
  font-style: italic;
}

/* 引用来源 */
.sources {
  margin-top: 6px;
  font-size: 12px;
  color: #666;
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
  align-items: center;
}
.source-tag {
  background: #eef2ff;
  color: #4b5563;
  padding: 2px 8px;
  border-radius: 10px;
}

/* 输入框区域 */
.input-bar {
  display: flex;
  gap: 10px;
  flex-shrink: 0;
}
.input {
  flex: 1;
  padding: 12px 14px;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  font-size: 14px;
  outline: none;
}
.send-btn {
  padding: 12px 18px;
  background: #f9f2e6;
  color: #333;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  margin-right: 15px;
}

.send-btn.isDark {
  background: #333;
  color: #fff;
}

/* 深色主题：聊天窗口整体 + 消息气泡 + 输入框 */
.chat-window.isDark {
  background: #1e1f26;
}
.chat-window.isDark .message.ai .msg-content {
  background: #2a2d35;
  color: #e5e7eb;
}
.chat-window.isDark .message.loading .msg-content {
  color: #9ca3af;
}
.chat-window.isDark .sources {
  color: #9ca3af;
}
.chat-window.isDark .source-tag {
  background: #2a2d35;
  color: #cbd5e1;
}
.chat-window.isDark .input {
  background: #2a2d35;
  border-color: #3a3d45;
  color: #e5e7eb;
}
.chat-window.isDark .input::placeholder {
  color: #8b8f98;
}
.send-btn:disabled {
  background: #93c5fd;
  cursor: not-allowed;
}
</style>
