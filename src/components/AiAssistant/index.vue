<template>
  <div class="ai-float-assistant">
    <!-- 悬浮球 -->
    <div class="float-ball" @click="toggleChat" :class="{ show: !isChatOpen }">
      <span class="icon">💬</span>
    </div>

    <!-- 聊天窗口 -->
    <div class="chat-window" v-show="isChatOpen" @click.stop>
      <div class="chat-header">
        <span>阿毕</span>
        <button class="close-btn" @click="toggleChat">×</button>
      </div>

      <div class="app-container">
        <!-- 左侧会话 -->
        <div class="sidebar">
          <button class="new-chat-btn" @click="createNewChat">
            ➕ 新建对话
          </button>
          <div class="session-list">
            <div
              v-for="item in sessionList"
              :key="item.sessionId"
              class="session-item"
              :class="{ active: item.sessionId === currentSessionId }"
              @click="switchSession(item.sessionId)"
            >
              <span class="title">{{
                item.sessionId.slice(-6) || "新对话"
              }}</span>
              <span class="del-btn" @click.stop="deleteSession(item.sessionId)">
                ×
              </span>
            </div>
          </div>
        </div>

        <!-- 右侧聊天 -->
        <div class="chat-container">
          <!-- 转人工服务按钮 -->
          <div style="margin-bottom: 10px">
            <button
              @click="switchToHuman"
              style="
                padding: 6px 12px;
                background: #a92a2a;
                color: #fff;
                border: none;
                border-radius: 6px;
              "
            >
              转人工服务
            </button>
            <button
              @click="switchToAi"
              style="
                padding: 6px 12px;
                background: #00b42a;
                color: #fff;
                border: none;
                border-radius: 6px;
                margin-left: 8px;
              "
            >
              切回AI助手
            </button>
          </div>

          <div class="message-list" ref="messageListEl">
            <!-- 用户消息 -->
            <div
              v-for="(msg, idx) in messages"
              :key="idx"
              class="message"
              :class="msg.role"
            >
              <!-- AI 消息 → 加头像 -->
              <div v-if="msg.role === 'ai'" class="avatar-box">
                <img src="@/assets/avatar.jpg" alt="AI" class="avatar" />
              </div>
              <div v-if="msg.content.trim()" class="msg-content">
                {{ msg.content }}
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
            <button @click="sendMessage" class="send-btn" :disabled="isLoading">
              {{ isLoading ? "发送中..." : "发送" }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch, nextTick } from "vue";
import { io } from "socket.io-client";

const props = defineProps({
  pageLocked: {
    type: Boolean,
    default: false,
  },
});
const emit = defineEmits(["toggle-page-lock"]);

const API_BASE = "https://api.bsq.asia";
// const API_BASE = "http://localhost:3000";
const input = ref("");
const messages = ref<any[]>([]);
const isLoading = ref(false);
const currentSessionId = ref("");
const isChatOpen = ref(false);
const sessionList = ref<any[]>([]);
const messageListEl = ref<HTMLElement | null>(null);
const sessionScrollMap = ref<Record<string, number>>({});
let es: EventSource | null = null;

// ====================== WebSocket 实时聊天 ======================
const socket = io("https://api.bsq.asia");
// const socket = io("http://localhost:3000");
onMounted(() => {
  socket.on("newMessage", () => {
    if (currentSessionId.value) {
      refreshChatHistory();
    }
  });
});

onUnmounted(() => {
  socket.disconnect();
  if (es) es.close();
});

watch(currentSessionId, (newId) => {
  if (newId) {
    socket.emit("joinSession", newId);
  }
});
// ==============================================================================

// 开关聊天 + 控制父组件禁止滚动
const toggleChat = () => {
  isChatOpen.value = !isChatOpen.value;
  emit("toggle-page-lock", !props.pageLocked);
};

// 初始化
onMounted(async () => {
  try {
    await loadAllSessions();
    const lastId = localStorage.getItem("lastSessionId");
    if (
      lastId &&
      Array.isArray(sessionList.value) &&
      sessionList.value.some((s) => s.sessionId === lastId)
    ) {
      switchSession(lastId);
    } else if (
      Array.isArray(sessionList.value) &&
      sessionList.value.length > 0
    ) {
      switchSession(sessionList.value[0].sessionId);
    } else {
      createNewChat();
    }
  } catch (err) {
    console.error("初始化失败", err);
  }
});

// 加载会话
async function loadAllSessions() {
  try {
    const res = await fetch(`${API_BASE}/chat/sessions`);
    let data = await res.json();
    sessionList.value =
      data?.data && Array.isArray(data.data)
        ? data.data
        : Array.isArray(data)
          ? data
          : [];
  } catch (err) {
    sessionList.value = [];
  }
}

// 新建
function createNewChat() {
  const newId = "session_" + Date.now();
  sessionList.value.unshift({
    sessionId: newId,
    title: `对话 ${sessionList.value.length + 1}`,
  });
  switchSession(newId);
  setTimeout(async () => {
    await sendWelcomeMessage();
  }, 300);
}

// 欢迎语
async function sendWelcomeMessage() {
  if (isLoading.value) return;
  if (es) es.close();

  isLoading.value = true;

  const url = `${API_BASE}/chat/stream?sessionId=${currentSessionId.value}&message=${encodeURIComponent("问世间情为何物？")}`;

  try {
    es = new EventSource(url);
    es.onmessage = (e) => {
      if (e.data === "[DONE") {
        es?.close();
        isLoading.value = false;
        loadAllSessions();
        return;
      }
      const last = messages.value[messages.value.length - 1];
      last?.role === "ai"
        ? (last.content += e.data)
        : messages.value.push({ role: "ai", content: e.data });
    };
    es.onerror = () => {
      es?.close();
      isLoading.value = false;
    };
  } catch (err) {
    isLoading.value = false;
  }
}

// 删除会话
async function deleteSession(sessionId: string) {
  if (!confirm("确定删除该会话吗？删除后不可恢复！")) return;
  try {
    await fetch(`${API_BASE}/chat/delete/${sessionId}`, { method: "DELETE" });
    await loadAllSessions();
    if (currentSessionId.value === sessionId) {
      sessionList.value.length
        ? switchSession(sessionList.value[0].sessionId)
        : createNewChat();
    }
  } catch (err) {}
}

// 转人工
const switchToHuman = async () => {
  await fetch(`${API_BASE}/chat/switch-human/${currentSessionId.value}`);
  alert("已切换为人工服务，客服将尽快回复您");
};

const switchToAi = async () => {
  await fetch(`${API_BASE}/chat/switch-ai/${currentSessionId.value}`);
  alert("已切回AI助手，将自动为您解答");
};

// 切换会话
async function switchSession(id: string) {
  try {
    if (currentSessionId.value && messageListEl.value) {
      sessionScrollMap.value[currentSessionId.value] =
        messageListEl.value.scrollTop;
    }

    if (es) {
      es.close();
      es = null;
    }

    currentSessionId.value = id;
    localStorage.setItem("lastSessionId", id);
    messages.value = [];
    isLoading.value = false;

    await refreshChatHistory();

    nextTick(() => {
      if (messageListEl.value) {
        const saveTop = sessionScrollMap.value[id] || 99999;
        messageListEl.value.scrollTop = saveTop;
      }
    });
  } catch (err) {
    console.error("切换会话失败", err);
  }
}

// ====================== 刷新聊天记录 ======================
async function refreshChatHistory() {
  const id = currentSessionId.value;
  if (!id) return;

  const res = await fetch(`${API_BASE}/chat/history/${id}`);
  const history = await res.json();
  const realHistory = Array.isArray(history) ? history : history.data || [];

  const list: any[] = [];
  realHistory.forEach((item) => {
    if (item.userMessage)
      list.push({ role: "user", content: item.userMessage });
    if (item.aiResponse) list.push({ role: "ai", content: item.aiResponse });
  });
  messages.value = list;

  nextTick(() => {
    if (messageListEl.value) {
      messageListEl.value.scrollTop = messageListEl.value.scrollHeight;
    }
  });
}
// ======================================================================

// 发送消息
const sendMessage = async () => {
  if (!input.value.trim() || isLoading.value) return;
  if (es) es.close();

  const userMsg = input.value.trim();

  // 查询是否人工模式
  const statusRes = await fetch(
    `${API_BASE}/chat/session-status/${currentSessionId.value}`,
  );
  const statusData = await statusRes.json();

  if (statusData.status === "human") {
    messages.value.push({ role: "user", content: userMsg });
    input.value = "";

    await fetch(`${API_BASE}/chat/user-send-message`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        sessionId: currentSessionId.value,
        message: userMsg,
      }),
    });

    return;
  }

  // AI 模式逻辑
  messages.value.push({ role: "user", content: userMsg });
  input.value = "";
  isLoading.value = true;

  const url = `${API_BASE}/chat/stream?sessionId=${currentSessionId.value}&message=${encodeURIComponent(userMsg)}`;

  try {
    es = new EventSource(url);
    es.onmessage = (e) => {
      if (e.data === "[DONE") {
        es?.close();
        isLoading.value = false;
        loadAllSessions();
        nextTick(() => {
          messageListEl.value!.scrollTop = messageListEl.value!.scrollHeight;
        });
        return;
      }
      const last = messages.value[messages.value.length - 1];
      last?.role === "ai"
        ? (last.content += e.data)
        : messages.value.push({ role: "ai", content: e.data });

      nextTick(() => {
        if (messageListEl.value) {
          messageListEl.value.scrollTop = messageListEl.value.scrollHeight;
        }
      });
    };
    es.onerror = () => {
      es?.close();
      isLoading.value = false;
    };
  } catch (err) {
    isLoading.value = false;
  }
};
</script>

<style scoped>
/* 全局容器 */
.ai-float-assistant {
  position: fixed;
  right: 20px;
  bottom: 20px;
  z-index: 9999;
}

/* 悬浮球 */
.float-ball {
  width: 60px;
  height: 60px;
  border-radius: 30%;
  background: #308bc4;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 24px;
  cursor: pointer;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  transition: all 0.3s ease;
}
.float-ball.show {
  display: flex;
}
.float-ball:hover {
  transform: scale(1.05);
}

/* 聊天窗口 - 核心修复：手机端全屏 + 键盘适配 */
.chat-window {
  position: fixed;
  right: 0;
  bottom: 0;
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

/* PC端恢复圆角大小 */
@media (min-width: 768px) {
  .chat-window {
    width: 700px;
    height: 650px;
    border-radius: 16px;
    left: auto;
    top: auto;
  }
}

.chat-header {
  padding: 14px 16px;
  background: #8e0000;
  color: white;
  font-weight: 500;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
}
.close-btn {
  background: none;
  border: none;
  color: white;
  font-size: 20px;
  cursor: pointer;
  line-height: 1;
}

/* 主内容区 */
.app-container {
  display: flex;
  flex: 1;
  overflow: hidden;
}

/* 侧边栏 */
.sidebar {
  width: 160px;
  background: #f7f8fa;
  border-right: 1px solid #eee;
  padding: 12px 8px;
  display: flex;
  flex-direction: column;
  gap: 8px;
  flex-shrink: 0;
}
/* 手机端侧边栏缩小 */
@media (max-width: 768px) {
  .sidebar {
    width: 120px;
  }

}

.new-chat-btn {
  padding: 8px 10px;
  background: #a92a2a;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 12px;
}
.session-list {
  flex: 1;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 4px;
  scrollbar-width: none;
  -ms-overflow-style: none;
}
.session-list::-webkit-scrollbar {
  display: none;
}
.session-item {
  padding: 8px 10px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  white-space: nowrap;
  overflow: hidden;
}
.session-item .title {
  overflow: hidden;
  text-overflow: ellipsis;
}
.session-item .del-btn {
  color: #ff4444;
  font-weight: bold;
  font-size: 14px;
  width: 16px;
  height: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: #fbeaea;
  opacity: 0.6;
}
.session-item .del-btn:hover {
  opacity: 1;
  background: #ffd6d6;
}
.session-item:hover {
  background: #eef2ff;
}
.session-item.active {
  background: #dbeafe;
  font-weight: 500;
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
.msg-content {
  padding: 10px 14px;
  border-radius: 14px;
  font-size: 14px;
  line-height: 1.5;
  word-wrap: break-word;
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

/* 输入框区域 - 防止键盘顶飞 */
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
  background: #8e0000;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 14px;
  margin-right:15px;
}
.send-btn:disabled {
  background: #93c5fd;
  cursor: not-allowed;
}
</style>