<script setup>
import { ref, onMounted } from 'vue'
const emit = defineEmits(['back', 'request-login'])
const props = defineProps(['user'])

const loading = ref(false)
const error = ref('')
const threads = ref([])
const detail = ref(null)
const detailLoading = ref(false)
const detailError = ref('')
const showDetail = ref(false)
const searchKeyword = ref('')
const searchLoading = ref(false)

const goHome = () => emit('back')
const triggerLogin = () => emit('request-login')

const openHome = () => {
  location.hash = '#/'
}

const openForum = () => {
  location.hash = '#/forum'
}

const fetchSummary = async () => {
  loading.value = true
  error.value = ''
  try {
    const res = await fetch('/forum/getForumSummaryInfo')
    const json = await res.json()
    if (json && json.code === 0 && Array.isArray(json.data)) {
      threads.value = json.data.map(x => ({
        id: x.id,
        title: x.title,
        summary: x.summary,
        username: x.username,
        publishTime: x.publishTime,
        tags: x.tags
      }))
    } else {
      error.value = json?.message || '加载失败'
    }
  } catch (e) {
    error.value = '网络错误或服务不可用'
  } finally {
    loading.value = false
  }
}

const searchPosts = async () => {
  if (!searchKeyword.value.trim()) return
  
  searchLoading.value = true
  error.value = ''
  try {
    const res = await fetch('/forum/searchForumPost', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ keyword: searchKeyword.value.trim() })
    })
    const json = await res.json()
    if (json && json.code === 0 && Array.isArray(json.data)) {
      threads.value = json.data.map(x => ({
        id: x.id,
        title: x.title,
        summary: x.summary,
        username: x.username,
        publishTime: x.publishTime,
        tags: x.tags
      }))
    } else {
      error.value = json?.message || '搜索失败'
    }
  } catch (e) {
    error.value = '网络错误或服务不可用'
  } finally {
    searchLoading.value = false
  }
}

const viewDetail = async (id) => {
  detail.value = null
  detailError.value = ''
  const token = localStorage.getItem('token')
  if (!token) {
    detailError.value = '需要登录后查看详情'
    emit('request-login')
    return
  }
  detailLoading.value = true
  try {
    const res = await fetch(`/forum/getForumInfoById?id=${encodeURIComponent(id)}`, {
      headers: { 'UserToken': token }
    })
    const json = await res.json()
    if (json && json.code === 0 && json.data) {
      detail.value = json.data
      showDetail.value = true
    } else {
      detailError.value = json?.message || '加载详情失败'
    }
  } catch (e) {
    detailError.value = '网络错误或服务不可用'
  } finally {
    detailLoading.value = false
  }
}

onMounted(fetchSummary)
</script>

<template>
  <div class="forum">
    <header class="masthead">
      <div class="logo-lockup">
        <span class="wordmark">RE/1999</span>
        <p>Return to Future</p>
      </div>
      <nav>
        <a href="#" @click.prevent="openHome">Home</a>
        <a href="#">News</a>
        <a href="#">Character</a>
        <a href="#">Backstory</a>
        <a href="#">Gallery</a>
        <a href="#">Download</a>
        <a href="#" @click.prevent="openForum">论坛</a>
      </nav>
      <div style="display:flex; gap:0.8rem; align-items:center; flex-wrap:wrap;">
        <button class="pill">预约测试</button>
        <button class="pill" @click="triggerLogin">{{ user ? `已登录：${user.username}` : '登录' }}</button>
      </div>
    </header>

    <section class="forum-hero">
      <h1>论坛</h1>
      <p>交流雨夜与旧时代的故事，分享你的见解。</p>
      <div class="search-container">
        <div class="search-box">
          <svg class="search-icon" width="20" height="20" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M11 19C15.4183 19 19 15.4183 19 11C19 6.58172 15.4183 3 11 3C6.58172 3 3 6.58172 3 11C3 15.4183 6.58172 19 11 19Z" stroke="rgba(236, 243, 247, 0.6)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M21 21L16.65 16.65" stroke="rgba(236, 243, 247, 0.6)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          <input 
            type="text" 
            placeholder="搜索帖子..." 
            v-model="searchKeyword"
            @keyup.enter="searchPosts"
            class="search-input"
          >
          <button @click="searchPosts" :disabled="searchLoading" class="search-button">
            <span v-if="searchLoading" class="loading-dots">
              <span></span>
              <span></span>
              <span></span>
            </span>
            <span v-else>搜索</span>
          </button>
        </div>
      </div>
    </section>

    <section class="thread-list">
      <p v-if="loading">加载中...</p>
      <p v-else-if="error" class="error">{{ error }}</p>
      <article v-else v-for="t in threads" :key="t.id" class="thread">
        <h3>{{ t.title }}</h3>
        <p class="meta">{{ t.username }} · {{ t.publishTime }}</p>
        <p>{{ t.summary }}</p>
        <p class="tags" v-if="t.tags">{{ t.tags }}</p>
        <button class="btn ghost" @click="viewDetail(t.id)">查看详情</button>
      </article>
    </section>

    <div v-if="showDetail" class="modal-mask" @click.self="showDetail=false">
      <div class="modal">
        <h3>{{ detail?.title }}</h3>
        <p class="meta">{{ detail?.username }} · {{ detail?.publishTime }}</p>
        <pre class="content">{{ detail?.content }}</pre>
        <div class="actions">
          <button class="btn ghost" @click="showDetail=false">关闭</button>
        </div>
      </div>
    </div>

    <p v-if="detailError" class="error" style="margin-top:1rem;">{{ detailError }}</p>
  </div>
</template>

<style scoped>
.forum {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  padding: 2.5rem 0 4rem;
  color: #ecf3f7;
}

.masthead {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1.5rem;
  padding: 1rem 1.5rem;
  border-radius: 20px;
  border: 1px solid rgba(86, 173, 173, 0.2);
  background: rgba(6, 14, 24, 0.7);
  backdrop-filter: blur(18px);
}

.masthead nav {
  display: flex;
  gap: 1.5rem;
  align-items: center;
}

.masthead nav a {
  color: #ecf3f7;
  text-decoration: none;
  font-size: 0.95rem;
  transition: all 0.3s ease;
}

.masthead nav a:hover {
  color: #9bd7cb;
}

.search-container {
  width: 100%;
  max-width: 700px;
  margin-top: 1.5rem;
}

.search-box {
  display: flex;
  align-items: center;
  position: relative;
  width: 100%;
  background: rgba(12, 28, 42, 0.65);
  border: 1px solid rgba(86, 173, 173, 0.25);
  border-radius: 28px;
  padding: 0.35rem 0.5rem 0.35rem 1.25rem;
  backdrop-filter: blur(10px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.search-box:focus-within {
  border-color: rgba(155, 215, 203, 0.6);
  box-shadow: 0 0 0 3px rgba(155, 215, 203, 0.1), 0 4px 16px rgba(0, 0, 0, 0.2);
  transform: translateY(-2px);
}

.search-icon {
  flex-shrink: 0;
  margin-right: 0.75rem;
  opacity: 0.6;
  transition: opacity 0.3s ease;
}

.search-box:focus-within .search-icon {
  opacity: 1;
  stroke: rgba(155, 215, 203, 0.8);
}

.search-input {
  flex: 1;
  padding: 0.85rem 0;
  border: none;
  background: transparent;
  color: #ecf3f7;
  outline: none;
  font-size: 0.95rem;
  font-family: inherit;
}

.search-input::placeholder {
  color: rgba(209, 220, 221, 0.45);
  transition: all 0.3s ease;
}

.search-box:focus-within .search-input::placeholder {
  color: rgba(209, 220, 221, 0.6);
}

.search-button {
  margin-left: 0.75rem;
  padding: 0.8rem 1.8rem;
  border: none;
  border-radius: 24px;
  background: linear-gradient(135deg, rgba(214, 184, 116, 0.25), rgba(155, 215, 203, 0.15));
  color: #ecf3f7;
  cursor: pointer;
  font-size: 0.95rem;
  font-weight: 500;
  font-family: inherit;
  border: 1px solid rgba(214, 184, 116, 0.4);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.search-button:hover:not(:disabled) {
  background: linear-gradient(135deg, rgba(214, 184, 116, 0.35), rgba(155, 215, 203, 0.25));
  border-color: rgba(214, 184, 116, 0.6);
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.search-button:active:not(:disabled) {
  transform: translateY(0);
}

.search-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  background: rgba(12, 28, 42, 0.5);
  border-color: rgba(86, 173, 173, 0.3);
}

/* Loading dots animation */
.loading-dots {
  display: flex;
  gap: 0.25rem;
  align-items: center;
  justify-content: center;
}

.loading-dots span {
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: rgba(236, 243, 247, 0.8);
  animation: loading 1.4s ease-in-out infinite both;
}

.loading-dots span:nth-child(1) {
  animation-delay: -0.32s;
}

.loading-dots span:nth-child(2) {
  animation-delay: -0.16s;
}

@keyframes loading {
  0%, 80%, 100% {
    transform: scale(0);
    opacity: 0.5;
  }
  40% {
    transform: scale(1);
    opacity: 1;
  }
}

.logo-lockup {
  display: flex;
  flex-direction: column;
  line-height: 1.1;
}

.wordmark {
  font-family: 'Playfair Display', serif;
  font-size: 1.4rem;
  letter-spacing: 0.48em;
}

.forum-hero {
  border-radius: 32px;
  padding: 2rem;
  background: linear-gradient(135deg, rgba(5, 9, 18, 0.95), rgba(15, 36, 48, 0.9));
  border: 1px solid rgba(86, 173, 173, 0.2);
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  gap: 1rem;
}

.forum-hero .search-box {
  width: 100%;
}

.thread-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 1rem;
}

.thread {
  border-radius: 20px;
  padding: 1.4rem;
  background: rgba(5, 12, 22, 0.8);
  border: 1px solid rgba(86, 173, 173, 0.12);
}

.meta {
  color: rgba(209, 220, 221, 0.6);
  font-size: 0.85rem;
}

.tags {
  color: #9bd7cb;
  font-size: 0.85rem;
}

.pill {
  border: 1px solid rgba(214, 184, 116, 0.5);
  background: transparent;
  padding: 0.6rem 1.5rem;
  border-radius: 999px;
  color: inherit;
  cursor: pointer;
}

.btn {
  border-radius: 999px;
  padding: 0.65rem 1.4rem;
  font-size: 0.95rem;
  cursor: pointer;
  border: none;
}

.btn.ghost {
  border: 1px solid rgba(219, 228, 232, 0.35);
  background: transparent;
  color: white;
}

.error {
  color: #f0c17a;
}

.modal-mask {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal {
  width: min(680px, 92vw);
  background: rgba(9, 24, 34, 0.95);
  border: 1px solid rgba(86, 173, 173, 0.25);
  border-radius: 20px;
  padding: 1.2rem 1.6rem;
  color: #ecf3f7;
}

.content {
  white-space: pre-wrap;
  line-height: 1.8;
}

.actions { margin-top: 1rem; }
</style>
