<template>
  <div class="chat-container" ref="chatContainerRef">
    <div class="chat-messages">
      <template v-if="currentChat && currentChat.messages.length > 0">
        <div
          v-for="(message, msgIndex) in currentChat.messages"
          :key="msgIndex"
          class="message-wrapper"
          :class="{
            'user-message': message.sender === 'user',
            'ai-message': message.sender === 'ai',
          }"
        >
          <div class="message-avatar">
            <el-avatar
              :size="40"
              :src="message.sender === 'user' ? userAvatar : aiAvatar"
            ></el-avatar>
          </div>
          <div class="message-content">
            <div class="message-bubble">
              <div class="message-text" v-html="message.content"></div>
              <div v-if="message.file" class="message-file">
                <a :href="message.file.url" target="_blank">{{ message.file.name }}</a>
              </div>
            </div>
            <div class="message-time">{{ message.time }}</div>
          </div>
        </div>
      </template>
      <div v-else class="empty-chat">
        <div class="empty-chat-content">
          <el-icon :size="48"><ChatLineRound /></el-icon>
          <p>开始一个新的对话吧</p>
        </div>
      </div>
    </div>
  </div>

  <div class="input-container">
    <div class="input-wrapper">
      <div class="message-input">
        <el-input
          v-model="inputMessage"
          type="textarea"
          :rows="1"
          placeholder="输入消息..."
          @keyup.enter="sendMessage"
          :disabled="isSending"
          resize="none"
          class="modern-input"
        />
      </div>
      <div class="send-btn">
        <el-button type="text" class="attachment-btn" @click="triggerFileInput">
          <el-icon :size="24"><Paperclip /></el-icon>
        </el-button>
        <input type="file" ref="fileInput" style="display: none" @change="handleFileUpload" />
        <el-button
          type="primary"
          round
          @click="sendMessage"
          :disabled="!inputMessage.trim() || isSending"
          class="modern-send-btn"
        >
          <el-icon><Position /></el-icon>
        </el-button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, nextTick } from 'vue'
import { Position, Paperclip, ChatLineRound } from '@element-plus/icons-vue'

// 当前聊天索引
const currentChatIndex = ref(0)
const currentChat = computed(() => chatHistory.value[currentChatIndex.value] || null)

// 用户和AI头像
const userAvatar = ref('/src/assets/user-avatar.png')
const aiAvatar = ref('/src/assets/ai-avatar.png')

// 聊天容器引用，用于滚动
const chatContainerRef = ref(null)

const inputMessage = ref('')
const isSending = ref(false)

//输入文件
const fileInput = ref(null)

// 聊天历史数据
const chatHistory = ref([
  {
    id: 1,
    title: '关于人工智能的讨论',
    createdAt: Date.now(),
    messages: [
      {
        sender: 'ai',
        content: '你好！我是AI助手，有什么可以帮助你的吗？',
        time: formatTime(new Date()),
      },
      {
        sender: 'user',
        content: '你能告诉我更多关于人工智能的知识吗？',
        time: formatTime(new Date()),
      },
      {
        sender: 'ai',
        content:
          '人工智能（AI）是计算机科学的一个分支，致力于创建能够执行通常需要人类智能的任务的系统。',
        time: formatTime(new Date()),
      },
    ],
  },
  {
    id: 2,
    title: '美股跌了QAQ',
    createdAt: Date.now() - 24 * 60 * 60 * 1000,
    messages: [
      {
        sender: 'ai',
        content: '你好！我是AI助手，有什么可以帮助你的吗？',
        time: formatTime(new Date(Date.now() - 24 * 60 * 60 * 1000)),
      },
      {
        sender: 'user',
        content: '吃什么呢',
        time: formatTime(new Date(Date.now() - 24 * 60 * 60 * 1000)),
      },
      {
        sender: 'ai',
        content: '您有任何特定的食物领域想了解更多吗？',
        time: formatTime(new Date(Date.now() - 24 * 60 * 60 * 1000)),
      },
    ],
  },
  {
    id: 3,
    title: '12',
    createdAt: Date.now() - 7 * 24 * 60 * 60 * 1000,
    messages: [
      {
        sender: 'ai',
        content: '你好！我是AI助手，有什么可以帮助你的吗？',
        time: formatTime(new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)),
      },
      {
        sender: 'user',
        content: '吃什么呢',
        time: formatTime(new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)),
      },
      {
        sender: 'ai',
        content: '您有任何特定的食物领域想了解更多吗？',
        time: formatTime(new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)),
      },
    ],
  },
  {
    id: 4,
    title: '123',
    createdAt: Date.now() - 10 * 24 * 60 * 60 * 1000,
    messages: [
      {
        sender: 'ai',
        content: '你好！我是AI助手，有什么可以帮助你的吗？',
        time: formatTime(new Date(Date.now() - 10 * 24 * 60 * 60 * 1000)),
      },
      {
        sender: 'user',
        content: '吃什么呢',
        time: formatTime(new Date(Date.now() - 10 * 24 * 60 * 60 * 1000)),
      },
      {
        sender: 'ai',
        content: '您有任何特定的食物领域想了解更多吗？',
        time: formatTime(new Date(Date.now() - 10 * 24 * 60 * 60 * 1000)),
      },
    ],
  },
])

// 格式化时间
function formatTime(date) {
  const hours = date.getHours().toString().padStart(2, '0')
  const minutes = date.getMinutes().toString().padStart(2, '0')
  return `${hours}:${minutes}`
}

// 触发文件选择
function triggerFileInput() {
  fileInput.value.click()
}

// 处理文件上传
function handleFileUpload(event) {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    const userMsg = {
      sender: 'user',
      content: inputMessage.value || '已上传文件',
      time: formatTime(new Date()),
      file: {
        name: file.name,
        url: e.target.result, // Data URL，可用于预览，展示逻辑需要后续修改
        type: file.type,
      },
    }
    chatHistory.value[currentChatIndex.value].messages.push(userMsg)
    inputMessage.value = ''
    scrollToBottom()

    // 模拟 AI 回复
    isSending.value = true
    setTimeout(() => {
      const aiResponse = {
        sender: 'ai',
        content: `已收到你的文件：${file.name}，有什么我可以帮忙的吗？`,
        time: formatTime(new Date()),
      }
      chatHistory.value[currentChatIndex.value].messages.push(aiResponse)
      isSending.value = false
      scrollToBottom()
    }, 1000)
  }
  reader.readAsDataURL(file)
  event.target.value = '' // 清空 input 以便重复选择同一文件
}

// 发送消息
function sendMessage() {
  if (!inputMessage.value.trim() || isSending.value) return

  const userMsg = {
    sender: 'user',
    content: inputMessage.value,
    time: formatTime(new Date()),
  }

  chatHistory.value[currentChatIndex.value].messages.push(userMsg)
  const sentMessage = inputMessage.value
  inputMessage.value = ''
  scrollToBottom()

  isSending.value = true
  setTimeout(() => {
    const aiResponse = {
      sender: 'ai',
      content: generateAIResponse(sentMessage),
      time: formatTime(new Date()),
    }
    chatHistory.value[currentChatIndex.value].messages.push(aiResponse)
    isSending.value = false
    nextTick(() => {
      scrollToBottom()
    })
  }, 1000)
}

// 模拟 AI 回复
function generateAIResponse(message) {
  const responses = [
    `我理解你的问题是关于"${message}"。这是一个很有趣的话题！`,
    `关于"${message}"，我有几点想分享：首先，这个领域正在快速发展；其次，有很多新的研究正在进行中。`,
    `谢谢你的提问！"${message}"确实值得深入探讨。从技术角度来看，这涉及到多个领域的交叉。`,
    `我很高兴你问到"${message}"。这让我们有机会探讨一些前沿概念和应用场景。`,
  ]
  return responses[Math.floor(Math.random() * responses.length)]
}

// 滚动到底部
function scrollToBottom() {
  nextTick(() => {
    if (chatContainerRef.value) {
      chatContainerRef.value.scrollTop = chatContainerRef.value.scrollHeight
    }
  })
}
</script>

<style scoped>
.chat-container {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
  background-color: #f9f9f9;
}

.chat-messages {
  max-width: 900px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.message-wrapper {
  display: flex;
  gap: 12px;
  max-width: 80%;
}

.user-message {
  margin-left: auto;
  flex-direction: row-reverse;
}

.message-avatar {
  flex-shrink: 0;
}

.message-content {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.message-bubble {
  padding: 12px 16px;
  border-radius: 12px;
  background-color: #fff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  overflow-wrap: break-word;
  word-break: break-word;
}

.user-message .message-bubble {
  background-color: #e8f5fe;
  border-top-right-radius: 4px;
}

.ai-message .message-bubble {
  background-color: #fff;
  border-top-left-radius: 4px;
}

.message-text {
  font-size: 14px;
  line-height: 1.5;
}

.message-time {
  font-size: 12px;
  color: #909399;
  margin-left: 8px;
}

.message-file {
  margin-top: 8px;
}

.message-file a {
  color: #409eff;
  text-decoration: none;
}

.message-file a:hover {
  text-decoration: underline;
}

.empty-chat {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.empty-chat-content {
  text-align: center;
  color: #909399;
}

.empty-chat-content p {
  margin-top: 16px;
  font-size: 16px;
}

.input-container {
  height: 100px;
  padding: 16px 20px;
  background-color: #fff;
  border-top: 1px solid #e4e7ed;
  display: flex;
  align-items: center;
}

.input-wrapper {
  max-width: 900px;
  width: 100%;
  margin: 0 auto;
  display: flex;
  align-items: center;
  gap: 12px;
  background-color: #f5f7fa;
  padding: 8px 12px;
  border-radius: 24px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  transition: box-shadow 0.3s;
}

.input-wrapper:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.message-input {
  flex: 1;
}

.modern-input {
  border: none;
  background: transparent;
}

.modern-input :deep(.el-textarea__inner) {
  border: none;
  background: transparent;
  padding: 8px 0;
  font-size: 14px;
  color: #303133;
  box-shadow: none;
}

.modern-input :deep(.el-textarea__inner:focus) {
  outline: none;
}

.send-btn {
  flex-shrink: 0;
  display: flex;
  align-items: center;
  gap: 8px;
}

.attachment-btn {
  margin-right: 0;
  transition: color 0.3s;
}

.attachment-btn:hover {
  color: #409eff;
}

.modern-send-btn {
  padding: 10px 20px;
  font-size: 14px;
  transition: transform 0.2s;
}

.modern-send-btn:hover {
  transform: scale(1.05);
}
</style>
