<script setup lang="ts">
import { ref, nextTick, computed } from 'vue'

interface Message {
  role: 'user' | 'assistant'
  content: string
  displayedContent?: string
  typing?: boolean
}

const messages = ref<Message[]>([])
const input = ref('')
const isTyping = ref(false)
const chatContainer = ref<HTMLElement | null>(null)
const textarea = ref<HTMLTextAreaElement | null>(null)

const examplePrompts = [
  'Explain quantum computing in simple terms',
  'Got any creative ideas for a 10 year old\'s birthday?',
  'How do I make an HTTP request in JavaScript?',
  'What is the meaning of life?',
]

function scrollToBottom() {
  nextTick(() => {
    if (chatContainer.value) {
      chatContainer.value.scrollTo({
        top: chatContainer.value.scrollHeight,
        behavior: 'smooth',
      })
    }
  })
}

function autoResize() {
  if (textarea.value) {
    textarea.value.style.height = 'auto'
    textarea.value.style.height = Math.min(textarea.value.scrollHeight, 200) + 'px'
  }
}

function typewriterEffect(msg: Message, text: string) {
  let i = 0
  msg.displayedContent = ''
  msg.typing = true
  const interval = setInterval(() => {
    msg.displayedContent = (msg.displayedContent ?? '') + (text[i] ?? '')
    i++
    scrollToBottom()
    if (i >= text.length) {
      clearInterval(interval)
      msg.typing = false
    }
  }, 60)
}

async function sendMessage(prefill?: string) {
  const text = (prefill || input.value).trim()
  if (!text || isTyping.value) return

  messages.value.push({ role: 'user', content: text, displayedContent: text })
  input.value = ''
  if (textarea.value) {
    textarea.value.style.height = 'auto'
  }
  isTyping.value = true
  scrollToBottom()

  // Simulate thinking delay
  await new Promise((r) => setTimeout(r, 1000 + Math.random() * 1000))

  const reply: Message = { role: 'assistant', content: 'idk haha', displayedContent: '' }
  messages.value.push(reply)
  isTyping.value = false
  typewriterEffect(reply, 'idk haha')
  scrollToBottom()
}

const canSend = computed(() => input.value.trim().length > 0 && !isTyping.value)
</script>

<template>
  <div class="app">
    <aside class="sidebar">
      <div class="sidebar-top">
        <button class="new-chat" @click="messages = []">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="5" x2="12" y2="19"/><line x1="5" y1="12" x2="19" y2="12"/></svg>
          New chat
        </button>
      </div>
      <div class="sidebar-bottom">
        <div class="sidebar-brand">
          <span class="brand-icon">🤷</span>
          <span>idkGPT</span>
        </div>
      </div>
    </aside>

    <main class="chat-area">
      <!-- Header bar -->
      <div class="topbar">
        <span class="topbar-model">idkGPT <span class="model-badge">v0.1</span></span>
      </div>

      <div class="chat-messages" ref="chatContainer">
        <!-- Empty state -->
        <div v-if="messages.length === 0" class="empty-state">
          <div class="empty-logo">🤷</div>
          <h1>How can I help you today?</h1>
          <div class="example-grid">
            <button
              v-for="(prompt, i) in examplePrompts"
              :key="i"
              class="example-card"
              @click="sendMessage(prompt)"
            >
              <span class="example-text">{{ prompt }}</span>
              <svg class="example-arrow" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 12h14M12 5l7 7-7 7"/></svg>
            </button>
          </div>
        </div>

        <!-- Messages -->
        <template v-for="(msg, i) in messages" :key="i">
          <div class="message-row" :class="msg.role">
            <div class="message-inner">
              <div class="message-body">
                <div class="message-role">{{ msg.role === 'user' ? 'You' : 'idkGPT' }}</div>
                <div class="message-content">
                  {{ msg.displayedContent || msg.content }}
                  <span v-if="msg.typing" class="cursor-blink">|</span>
                </div>
              </div>
            </div>
          </div>
        </template>

        <!-- Typing indicator -->
        <div v-if="isTyping" class="message-row assistant">
          <div class="message-inner">
            <div class="message-body">
              <div class="message-role">idkGPT</div>
              <div class="message-content typing-indicator">
                <span class="dot"></span><span class="dot"></span><span class="dot"></span>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Input -->
      <div class="input-wrapper">
        <div class="input-area">
          <form @submit.prevent="sendMessage()" class="input-form">
            <textarea
              ref="textarea"
              v-model="input"
              placeholder="Message idkGPT..."
              rows="1"
              @input="autoResize"
              @keydown.enter.exact.prevent="sendMessage()"
            />
            <button type="submit" :disabled="!canSend" :class="{ active: canSend }">
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
                <path d="M6 12L12 6L18 12" stroke="currentColor" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"/>
                <path d="M12 6V18" stroke="currentColor" stroke-width="2.5" stroke-linecap="round"/>
              </svg>
            </button>
          </form>
        </div>
      </div>
    </main>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: #212121;
  color: #e3e3e3;
  height: 100vh;
  overflow: hidden;
}

#app {
  height: 100vh;
}

.app {
  display: flex;
  height: 100vh;
}

/* Custom scrollbar */
::-webkit-scrollbar {
  width: 6px;
}
::-webkit-scrollbar-track {
  background: transparent;
}
::-webkit-scrollbar-thumb {
  background: rgba(255,255,255,0.12);
  border-radius: 3px;
}
::-webkit-scrollbar-thumb:hover {
  background: rgba(255,255,255,0.2);
}

/* ───── Sidebar ───── */
.sidebar {
  width: 260px;
  background: #171717;
  display: flex;
  flex-direction: column;
  padding: 10px;
  border-right: 1px solid rgba(255,255,255,0.06);
}

.sidebar-top {
  padding-bottom: 10px;
}

.new-chat {
  width: 100%;
  padding: 10px 14px;
  border: 1px solid rgba(255,255,255,0.12);
  border-radius: 10px;
  background: transparent;
  color: #e3e3e3;
  font-size: 14px;
  font-family: inherit;
  cursor: pointer;
  text-align: left;
  display: flex;
  align-items: center;
  gap: 10px;
  transition: all 0.15s ease;
}

.new-chat:hover {
  background: rgba(255,255,255,0.08);
  border-color: rgba(255,255,255,0.2);
}

.sidebar-bottom {
  margin-top: auto;
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 14px;
  border-radius: 10px;
  font-size: 14px;
  font-weight: 600;
  color: rgba(255,255,255,0.6);
}

.brand-icon {
  font-size: 22px;
}

/* ───── Top bar ───── */
.topbar {
  padding: 14px 20px;
  border-bottom: 1px solid rgba(255,255,255,0.06);
  display: flex;
  align-items: center;
  background: #212121;
}

.topbar-model {
  font-size: 15px;
  font-weight: 600;
  display: flex;
  align-items: center;
  gap: 8px;
}

.model-badge {
  font-size: 11px;
  font-weight: 500;
  background: rgba(255,255,255,0.08);
  padding: 2px 8px;
  border-radius: 6px;
  color: rgba(255,255,255,0.5);
}

/* ───── Chat area ───── */
.chat-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: #212121;
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 24px 0 8px;
}

/* ───── Empty state ───── */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100%;
  gap: 16px;
  padding: 20px;
}

.empty-logo {
  font-size: 56px;
  margin-bottom: 8px;
  animation: float 3s ease-in-out infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}

.empty-state h1 {
  font-size: 28px;
  font-weight: 700;
  color: #e3e3e3;
}

.example-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 10px;
  max-width: 640px;
  width: 100%;
  margin-top: 16px;
}

.example-card {
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 14px;
  padding: 16px;
  color: #b4b4b4;
  font-size: 13.5px;
  line-height: 1.5;
  text-align: left;
  cursor: pointer;
  font-family: inherit;
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 12px;
  transition: all 0.15s ease;
}

.example-card:hover {
  background: rgba(255,255,255,0.08);
  border-color: rgba(255,255,255,0.15);
  color: #e3e3e3;
}

.example-arrow {
  flex-shrink: 0;
  opacity: 0;
  transition: opacity 0.15s ease;
  margin-top: 2px;
}

.example-card:hover .example-arrow {
  opacity: 0.6;
}

/* ───── Messages ───── */
.message-row {
  padding: 12px 24px;
  animation: fadeIn 0.3s ease;
  display: flex;
  max-width: 768px;
  margin: 0 auto;
  width: 100%;
}

.message-row.user {
  justify-content: flex-end;
}

.message-row.assistant {
  justify-content: flex-start;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(6px); }
  to { opacity: 1; transform: translateY(0); }
}

.message-inner {
  max-width: 75%;
}

.message-body {
  min-width: 0;
}

.message-row.user .message-body .message-content {
  background: #2f2f2f;
  border-radius: 18px 18px 4px 18px;
}

.message-row.assistant .message-body .message-content {
  background: #303048;
  border-radius: 18px 18px 18px 4px;
}

.message-role {
  font-size: 12px;
  font-weight: 600;
  margin-bottom: 4px;
  color: rgba(255,255,255,0.5);
}

.message-row.user .message-role {
  text-align: right;
}

.message-content {
  font-size: 15px;
  line-height: 1.6;
  color: #e3e3e3;
  word-break: break-word;
  padding: 10px 16px;
  display: inline-block;
}

.cursor-blink {
  animation: blink 0.8s step-end infinite;
  font-weight: 200;
  color: rgba(255,255,255,0.6);
}

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0; }
}

/* ───── Typing indicator ───── */
.typing-indicator {
  display: flex;
  gap: 5px;
  padding: 4px 0;
}

.dot {
  width: 7px;
  height: 7px;
  background: rgba(255,255,255,0.5);
  border-radius: 50%;
  animation: dotPulse 1.4s infinite ease-in-out both;
}

.dot:nth-child(1) { animation-delay: -0.32s; }
.dot:nth-child(2) { animation-delay: -0.16s; }
.dot:nth-child(3) { animation-delay: 0s; }

@keyframes dotPulse {
  0%, 80%, 100% { transform: scale(0.4); opacity: 0.4; }
  40% { transform: scale(1); opacity: 1; }
}

/* ───── Input area ───── */
.input-wrapper {
  padding: 0 24px 24px;
  background: linear-gradient(transparent, #212121 20%);
}

.input-area {
  max-width: 768px;
  margin: 0 auto;
  width: 100%;
}

.input-form {
  display: flex;
  align-items: flex-end;
  gap: 10px;
  background: #2f2f2f;
  border-radius: 16px;
  border: 1px solid rgba(255,255,255,0.08);
  padding: 12px 16px;
  transition: border-color 0.2s ease, box-shadow 0.2s ease;
}

.input-form:focus-within {
  border-color: rgba(255,255,255,0.18);
  box-shadow: 0 0 0 1px rgba(255,255,255,0.06);
}

.input-form textarea {
  flex: 1;
  background: transparent;
  border: none;
  color: #e3e3e3;
  font-size: 15px;
  line-height: 1.5;
  resize: none;
  outline: none;
  font-family: inherit;
  max-height: 200px;
}

.input-form textarea::placeholder {
  color: rgba(255,255,255,0.3);
}

.input-form button {
  background: #676767;
  border: none;
  border-radius: 10px;
  width: 34px;
  height: 34px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  color: #212121;
  transition: all 0.15s ease;
  flex-shrink: 0;
}

.input-form button:disabled {
  opacity: 0.25;
  cursor: not-allowed;
}

.input-form button.active {
  background: #e3e3e3;
  opacity: 1;
}

.input-form button.active:hover {
  background: #ffffff;
}

.disclaimer {
  text-align: center;
  font-size: 12px;
  color: rgba(255,255,255,0.25);
  margin-top: 10px;
}

/* ───── Responsive ───── */
@media (max-width: 768px) {
  .sidebar {
    display: none;
  }
  .example-grid {
    grid-template-columns: 1fr;
  }
  .message-inner {
    gap: 12px;
  }
  .message-row {
    padding: 16px 16px;
  }
  .input-wrapper {
    padding: 0 12px 16px;
  }
}
</style>
