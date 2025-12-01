<script setup>
import { ref, onMounted } from 'vue'
const emit = defineEmits(['back', 'request-login'])

const loading = ref(false)
const error = ref('')
const threads = ref([])
const detail = ref(null)
const detailLoading = ref(false)
const detailError = ref('')
const showDetail = ref(false)

const goHome = () => emit('back')

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
    const res = await fetch(`/forum/getForumDetail?id=${encodeURIComponent(id)}`, {
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
    <header class="forum-masthead">
      <div class="logo-lockup">
        <span class="wordmark">RE/1999</span>
        <p>Forum</p>
      </div>
      <button class="pill" @click="goHome">返回首页</button>
    </header>

    <section class="forum-hero">
      <h1>论坛</h1>
      <p>交流雨夜与旧时代的故事，分享你的见解。</p>
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

.forum-masthead {
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
