<template>
  <div class="sidebar" :class="{ 'sidebar-collapsed': sidebarCollapsed }">
    <div class="sidebar-header">
      <h3>历史对话</h3>
    </div>
    <div class="sidebar-content">
      <div class="conversation-list">
        <!-- 今天 -->
        <div v-if="todayChats.length" class="time-group">
          <div class="time-group-title">今天</div>
          <div
            v-for="(chat, index) in todayChats"
            :key="chat.id"
            class="conversation-item"
            :class="{ active: currentChatIndex === getGlobalIndex(chat) }"
            @click="switchChat(getGlobalIndex(chat))"
          >
            <el-tooltip :content="chat.title" placement="right" :show-after="1000">
              <div class="conversation-title">
                <el-icon><ChatRound /></el-icon>
                <span>{{ chat.title }}</span>
              </div>
            </el-tooltip>
            <div class="conversation-actions">
              <el-dropdown trigger="click" @command="handleCommand($event, getGlobalIndex(chat))">
                <el-icon><MoreFilled /></el-icon>
                <template #dropdown>
                  <el-dropdown-menu>
                    <el-dropdown-item command="rename">重命名</el-dropdown-item>
                    <el-dropdown-item command="delete" divided>删除</el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
            </div>
          </div>
        </div>

        <!-- 昨天 -->
        <div v-if="yesterdayChats.length" class="time-group">
          <div class="time-group-title">昨天</div>
          <div
            v-for="(chat, index) in yesterdayChats"
            :key="chat.id"
            class="conversation-item"
            :class="{ active: currentChatIndex === getGlobalIndex(chat) }"
            @click="switchChat(getGlobalIndex(chat))"
          >
            <el-tooltip :content="chat.title" placement="right" :show-after="1000">
              <div class="conversation-title">
                <el-icon><ChatRound /></el-icon>
                <span>{{ chat.title }}</span>
              </div>
            </el-tooltip>
            <div class="conversation-actions">
              <el-dropdown trigger="click" @command="handleCommand($event, getGlobalIndex(chat))">
                <el-icon><MoreFilled /></el-icon>
                <template #dropdown>
                  <el-dropdown-menu>
                    <el-dropdown-item command="rename">重命名</el-dropdown-item>
                    <el-dropdown-item command="delete" divided>删除</el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
            </div>
          </div>
        </div>

        <!-- 七天前 -->
        <div v-if="weekChats.length" class="time-group">
          <div class="time-group-title">七天前</div>
          <div
            v-for="(chat, index) in weekChats"
            :key="chat.id"
            class="conversation-item"
            :class="{ active: currentChatIndex === getGlobalIndex(chat) }"
            @click="switchChat(getGlobalIndex(chat))"
          >
            <el-tooltip :content="chat.title" placement="right" :show-after="1000">
              <div class="conversation-title">
                <el-icon><ChatRound /></el-icon>
                <span>{{ chat.title }}</span>
              </div>
            </el-tooltip>
            <div class="conversation-actions">
              <el-dropdown trigger="click" @command="handleCommand($event, getGlobalIndex(chat))">
                <el-icon><MoreFilled /></el-icon>
                <template #dropdown>
                  <el-dropdown-menu>
                    <el-dropdown-item command="rename">重命名</el-dropdown-item>
                    <el-dropdown-item command="delete" divided>删除</el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
            </div>
          </div>
        </div>

        <!-- 更早 -->
        <div v-if="olderChats.length" class="time-group">
          <div class="time-group-title">更早</div>
          <div
            v-for="(chat, index) in olderChats"
            :key="chat.id"
            class="conversation-item"
            :class="{ active: currentChatIndex === getGlobalIndex(chat) }"
            @click="switchChat(getGlobalIndex(chat))"
          >
            <el-tooltip :content="chat.title" placement="right" :show-after="1000">
              <div class="conversation-title">
                <el-icon><ChatRound /></el-icon>
                <span>{{ chat.title }}</span>
              </div>
            </el-tooltip>
            <div class="conversation-actions">
              <el-dropdown trigger="click" @command="handleCommand($event, getGlobalIndex(chat))">
                <el-icon><MoreFilled /></el-icon>
                <template #dropdown>
                  <el-dropdown-menu>
                    <el-dropdown-item command="rename">重命名</el-dropdown-item>
                    <el-dropdown-item command="delete" divided>删除</el-dropdown-item>
                  </el-dropdown-menu>
                </template>
              </el-dropdown>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="sidebar-footer">
      <el-button type="primary" @click="createNewChat" class="new-chat-btn">
        <el-icon><Plus /></el-icon>
        <span>新建对话</span>
      </el-button>
    </div>

    <!-- 重命名部分 -->
    <el-dialog
      v-model="renameDialogVisible"
      title="重命名对话"
      width="30%"
      :before-close="handleDialogClose"
    >
      <el-input v-model="newChatTitle" placeholder="请输入对话名称" />
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="renameDialogVisible = false">取消</el-button>
          <el-button type="primary" @click="confirmRename">确认</el-button>
        </span>
      </template>
    </el-dialog>
  </div>

  <!-- 主内容区 -->
  <div class="main-content">
    <div class="main-header">
      <div class="header-left">
        <div class="toggle-sidebar" @click="toggleSidebar">
          <el-icon><Fold v-if="!sidebarCollapsed" /><Expand v-else /></el-icon>
        </div>
        <div class="app-title">中南大学deepseek模型</div>
      </div>
      <div class="header-right">
        <el-button type="text">
          <el-icon><Setting /></el-icon>
        </el-button>
        <el-avatar :size="32" :src="userAvatar"></el-avatar>
      </div>
    </div>
    <!-- 对话部分 -->
    <Chat />
  </div>
</template>

<script setup>
import { ref, computed, nextTick, watch, onMounted } from 'vue'
import { ChatRound, Plus, MoreFilled, Setting, Fold, Expand } from '@element-plus/icons-vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import Chat from './Chat.vue'
// 用户和AI头像
const userAvatar = ref('/src/assets/user-avatar.png')
const aiAvatar = ref('/src/assets/ai-avatar.png')

// 侧边栏状态
const sidebarCollapsed = ref(false)
const toggleSidebar = () => {
  sidebarCollapsed.value = !sidebarCollapsed.value
}
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

const currentChatIndex = ref(0)
const currentChat = computed(() => chatHistory.value[currentChatIndex.value] || null)
const chatContainerRef = ref(null)
const renameDialogVisible = ref(false)
const chatToRenameIndex = ref(null)
const newChatTitle = ref('')

const todayChats = computed(() => {
  const now = new Date()
  const startOfToday = new Date(now.getFullYear(), now.getMonth(), now.getDate())
  return chatHistory.value.filter((chat) => new Date(chat.createdAt) >= startOfToday)
})

const yesterdayChats = computed(() => {
  const now = new Date()
  const startOfToday = new Date(now.getFullYear(), now.getMonth(), now.getDate())
  const startOfYesterday = new Date(startOfToday)
  startOfYesterday.setDate(startOfYesterday.getDate() - 1)
  return chatHistory.value.filter((chat) => {
    const chatDate = new Date(chat.createdAt)
    return chatDate >= startOfYesterday && chatDate < startOfToday
  })
})

const weekChats = computed(() => {
  const now = new Date()
  const startOfToday = new Date(now.getFullYear(), now.getMonth(), now.getDate())
  const startOfYesterday = new Date(startOfToday)
  startOfYesterday.setDate(startOfYesterday.getDate() - 1)
  const startOfWeekAgo = new Date(startOfToday)
  startOfWeekAgo.setDate(startOfWeekAgo.getDate() - 7)
  return chatHistory.value.filter((chat) => {
    const chatDate = new Date(chat.createdAt)
    return chatDate >= startOfWeekAgo && chatDate < startOfYesterday
  })
})

const olderChats = computed(() => {
  const now = new Date()
  const startOfWeekAgo = new Date(now.getFullYear(), now.getMonth(), now.getDate())
  startOfWeekAgo.setDate(startOfWeekAgo.getDate() - 7)
  return chatHistory.value.filter((chat) => new Date(chat.createdAt) < startOfWeekAgo)
})

const getGlobalIndex = (chat) => {
  return chatHistory.value.findIndex((item) => item.id === chat.id)
}

function formatTime(date) {
  const hours = date.getHours().toString().padStart(2, '0')
  const minutes = date.getMinutes().toString().padStart(2, '0')
  return `${hours}:${minutes}`
}

function createNewChat() {
  const newChat = {
    id: Date.now(),
    title: `新对话 ${chatHistory.value.length + 1}`,
    createdAt: Date.now(),
    messages: [
      {
        sender: 'ai',
        content: '你好！我是AI助手，有什么可以帮助你的吗？',
        time: formatTime(new Date()),
      },
    ],
  }
  chatHistory.value.push(newChat)
  currentChatIndex.value = chatHistory.value.length - 1
  scrollToBottom()
}

function switchChat(index) {
  currentChatIndex.value = index
  nextTick(() => scrollToBottom())
}

function handleCommand(command, index) {
  if (command === 'rename') {
    chatToRenameIndex.value = index
    newChatTitle.value = chatHistory.value[index].title
    renameDialogVisible.value = true
  } else if (command === 'delete') {
    ElMessageBox.confirm('确定要删除这个对话吗？此操作不可撤销。', '删除对话', {
      confirmButtonText: '确定',
      cancelButtonText: '取消',
      type: 'warning',
    })
      .then(() => deleteChat(index))
      .catch(() => {})
  }
}

function deleteChat(index) {
  chatHistory.value.splice(index, 1)
  if (chatHistory.value.length === 0) {
    createNewChat()
  } else if (currentChatIndex.value >= chatHistory.value.length) {
    currentChatIndex.value = chatHistory.value.length - 1
  }
  ElMessage({ type: 'success', message: '对话已删除' })
}

function confirmRename() {
  if (newChatTitle.value.trim()) {
    chatHistory.value[chatToRenameIndex.value].title = newChatTitle.value.trim()
    renameDialogVisible.value = false
    ElMessage({ type: 'success', message: '对话已重命名' })
  } else {
    ElMessage({ type: 'warning', message: '名称不能为空' })
  }
}

function handleDialogClose() {
  renameDialogVisible.value = false
}

function scrollToBottom() {
  nextTick(() => {
    if (chatContainerRef.value) {
      chatContainerRef.value.scrollTop = chatContainerRef.value.scrollHeight
    }
  })
}

watch(
  () => currentChat.value?.messages.length,
  () => scrollToBottom(),
)

onMounted(() => scrollToBottom())
</script>

<style scoped>
.sidebar {
  width: 280px;
  height: 100%;
  display: flex;
  flex-direction: column;
  background-color: #fff;
  border-right: 1px solid #e4e7ed;
  transition: width 0.3s;
  overflow: hidden;
}

.sidebar-collapsed {
  width: 0;
}

.sidebar-header {
  padding: 16px;
  border-bottom: 1px solid #e4e7ed;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.sidebar-header h3 {
  margin: 0;
  color: #303133;
  font-size: 16px;
  font-weight: 600;
}

.sidebar-content {
  flex: 1;
  overflow-y: auto;
  padding: 8px 0;
}

.conversation-list {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.time-group {
  margin-bottom: 16px;
}

.time-group-title {
  padding: 8px 16px;
  color: #606266;
  font-size: 14px;
  font-weight: 500;
  border-bottom: 1px solid #e4e7ed;
}

.conversation-item {
  padding: 10px 16px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  cursor: pointer;
  transition: background-color 0.3s;
  border-radius: 4px;
  margin: 0 4px;
}

.conversation-item:hover {
  background-color: #f5f7fa;
}

.conversation-item.active {
  background-color: #ecf5ff;
}

.conversation-title {
  display: flex;
  align-items: center;
  gap: 8px;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
  flex: 1;
}

.conversation-title span {
  font-size: 14px;
}

.conversation-actions {
  opacity: 0;
  transition: opacity 0.3s;
}

.conversation-item:hover .conversation-actions {
  opacity: 1;
}

.conversation-item.active .conversation-actions {
  opacity: 1;
}

.sidebar-footer {
  padding: 16px;
  border-top: 1px solid #e4e7ed;
}

.new-chat-btn {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  margin-top: 20px;
  gap: 12px;
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  height: 100%;
  overflow: hidden;
}

.main-header {
  height: 60px;
  padding: 0 20px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: #fff;
  border-bottom: 1px solid #e4e7ed;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.05);
  z-index: 10;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 16px;
}

.toggle-sidebar {
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 32px;
  height: 32px;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.toggle-sidebar:hover {
  background-color: #f5f7fa;
}

.app-title {
  font-size: 18px;
  font-weight: 600;
  color: #303133;
}

.header-right {
  display: flex;
  align-items: center;
  gap: 16px;
}
</style>
