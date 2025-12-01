<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import ForumPage from './pages/ForumPage.vue'

const navLinks = ['Home', 'News', 'Character', 'Backstory', 'Gallery', 'Download', '论坛']

const newsList = [
  {
    tag: '活动',
    title: '暴雨档案调律计划开启',
    copy: '跟随调律者踏上伦敦雨夜巡礼，解锁限定插画与语音片段。',
    date: '2025-11-21'
  },
  {
    tag: '公告',
    title: '「1999最后一天」剧情复刻',
    copy: '重温暴雨降临世界瞬间的惊鸿一瞥，追加全新交互段落。',
    date: '2025-11-15'
  },
  {
    tag: '新闻',
    title: '官方原声专辑发布',
    copy: '将怀旧黑胶与弦乐再制，记录雨滴倒流的每一次脉搏。',
    date: '2025-11-07'
  }
]

const characterCards = [
  {
    name: '索姆尼亚 Somnia',
    class: '唤雨者',
    brief: '操纵倒悬雨滴的御职角色，擅长带来高额领域加成。',
    badge: 'SSR'
  },
  {
    name: '十六夜 Izayoi',
    class: '抄录员',
    brief: '背着老式收音机记录雨幕咒文，技能如诗句逐行落下。',
    badge: 'SR'
  },
  {
    name: '维里蒂 Verity',
    class: '守夜人',
    brief: '以古董探照灯劈开雾气，提供强力护盾与异常抵御。',
    badge: 'SSR'
  }
]

const worldChapters = [
  {
    title: '暴雨降临',
    detail: '1999 年最后一天，地面溢起积水，雨滴倒悬向天——一切从伦敦开始。'
  },
  {
    title: '旧时代剥落',
    detail: '行人、建筑在雨中剥落溶解，时间像薄纸般被揭下，露出崭新的旧篇章。'
  },
  {
    title: '调律者的邀约',
    detail: '只有你未被侵蚀，成为沟通暴雨与现实的信使，搜集回声图鉴。'
  }
]

const galleryShots = [
  { caption: '雨夜广播站', note: '霓虹与蒸汽交错的电台，播放 1999 的倒带声。' },
  { caption: '调律者站', note: '角色海报墙与铭刻涂鸦并置，像翻阅平行叙事。' },
  { caption: '逆潮剧场', note: '金属浮雕舞台搭配墨蓝幕布，剧情在此首演。' }
]

const videoSlides = [
  {
    title: '暴雨序章',
    desc: '粒子化雨滴与倒置城市交织，带你进入 1999 的黎明前夜。',
    url: 'https://storage.googleapis.com/coverr-main/mp4/Mt_Baker.mp4'
  },
  {
    title: '角色调律',
    desc: '调律者立绘与技能特写穿插蒸汽齿轮，让登场更具仪式感。',
    url: 'https://storage.googleapis.com/coverr-main/mp4/Night-Sky.mp4'
  },
  {
    title: '异相漫游',
    desc: '沿着荧光雨滴穿梭伦敦街头，感受旧时代剥落的瞬间。',
    url: 'https://storage.googleapis.com/coverr-main/mp4/Vancouver.mp4'
  }
]

const activeSlide = ref(0)
let slideTimer

const switchSlide = (direction = 1) => {
  activeSlide.value = (activeSlide.value + direction + videoSlides.length) % videoSlides.length
}

onMounted(() => {
  slideTimer = setInterval(() => switchSlide(1), 6000)
  window.addEventListener('hashchange', applyRoute)
  applyRoute()
})

onBeforeUnmount(() => {
  clearInterval(slideTimer)
  window.removeEventListener('hashchange', applyRoute)
})

const raindrops = Array.from({ length: 24 }, (_, idx) => ({
  id: idx,
  delay: (idx % 6) * 0.4,
  duration: 2.8 + (idx % 5) * 0.35,
  left: `${(idx * 7) % 100}%`
}))

const currentPage = ref('home')
const applyRoute = () => {
  currentPage.value = location.hash.startsWith('#/forum') ? 'forum' : 'home'
}
const showLogin = ref(false)
const loginForm = ref({ username: '', password: '' })
const user = ref(null)
const loginLoading = ref(false)
const loginError = ref('')

const openForum = () => {
  location.hash = '#/forum'
  applyRoute()
}

const openHome = () => {
  location.hash = '#/'
  applyRoute()
}

const triggerLogin = () => {
  showLogin.value = true
  loginError.value = ''
}

const closeLogin = () => {
  showLogin.value = false
}

const doLogin = async () => {
  loginError.value = ''
  if (!loginForm.value.username || !loginForm.value.password) {
    loginError.value = '请输入账号和密码'
    return
  }
  loginLoading.value = true
  try {
    const res = await fetch('/login/user', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        username: loginForm.value.username,
        password: loginForm.value.password
      })
    })
    const json = await res.json()
    if (json && json.code === 0 && json.data) {
      user.value = json.data
      if (json.data.token) localStorage.setItem('token', json.data.token)
      showLogin.value = false
    } else {
      loginError.value = json?.message || '登录失败'
    }
  } catch (e) {
    loginError.value = '网络错误或服务不可用'
  } finally {
    loginLoading.value = false
  }
}
</script>

<template>
  <ForumPage v-if="currentPage === 'forum'" @back="openHome" @request-login="triggerLogin" />
  <div v-else class="site">
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

    <section class="hero">
      <div class="rain-layer">
        <span
          v-for="drop in raindrops"
          :key="drop.id"
          class="raindrop"
          :style="{
            animationDuration: `${drop.duration}s`,
            animationDelay: `${drop.delay}s`,
            left: drop.left
          }"
        />
      </div>
      <div class="hero-copy">
        <p class="eyebrow">1999 · 最后的雨</p>
        <h1>当暴雨向天空倾泻，倒悬的雨滴是你的指引</h1>
        <p>
          1999 年最后一天，世界被一场逆向暴雨吞没。地面溢起积水，行人和墙壁在雨中剥落，
          除你之外的一切都被卷入新的旧时代。现在，作为调律者，去追寻隐藏在层层雨幕后的秘密。
        </p>
        <div class="hero-actions">
          <button class="btn primary">立即进入</button>
          <button class="btn ghost">观看预告</button>
        </div>
      </div>
      <div class="hero-card">
        <p class="tag">暴雨时刻</p>
        <h3>23:59 · 1999.12.31</h3>
        <p>你的指尖碰到飞升的雨滴，时间开始倒流。</p>
        <div class="hero-stat">
          <div>
            <span>异相同步率</span>
            <strong>97%</strong>
          </div>
          <div>
            <span>调律节点</span>
            <strong>12</strong>
          </div>
        </div>
      </div>
      <div class="hero-glow"></div>
    </section>

    <section class="media panel">
      <header>
        <p class="eyebrow">GALLERY</p>
        <h2>暴雨影像轮播</h2>
      </header>
      <div class="carousel">
        <button class="carousel-btn prev" @click="switchSlide(-1)">‹</button>
        <div class="carousel-window">
          <article
            v-for="(slide, index) in videoSlides"
            :key="slide.title"
            class="carousel-slide"
            :class="{ active: index === activeSlide }"
          >
            <video :src="slide.url" autoplay muted loop playsinline></video>
            <div class="slide-copy">
              <h3>{{ slide.title }}</h3>
              <p>{{ slide.desc }}</p>
            </div>
          </article>
        </div>
        <button class="carousel-btn next" @click="switchSlide(1)">›</button>
      </div>
      <div class="carousel-dots">
        <button
          v-for="(slide, index) in videoSlides"
          :key="slide.title"
          :class="{ active: index === activeSlide }"
          @click="activeSlide = index"
        ></button>
      </div>
    </section>

    <section class="news">
      <header>
        <p class="eyebrow">NEWS</p>
        <h2>最新情报</h2>
        <a href="#">全部资讯 →</a>
      </header>
      <div class="news-grid">
        <article v-for="item in newsList" :key="item.title">
          <span class="chip">{{ item.tag }}</span>
          <h3>{{ item.title }}</h3>
          <p>{{ item.copy }}</p>
          <time>{{ item.date }}</time>
        </article>
      </div>
    </section>

    <section class="characters">
      <header>
        <p class="eyebrow">CHARACTER</p>
        <h2>调律者档案</h2>
      </header>
      <div class="character-grid">
        <article v-for="card in characterCards" :key="card.name">
          <span class="badge">{{ card.badge }}</span>
          <h3>{{ card.name }}</h3>
          <p class="role">{{ card.class }}</p>
          <p>{{ card.brief }}</p>
        </article>
      </div>
    </section>

    <section class="world">
      <div class="world-bg"></div>
      <header>
        <p class="eyebrow">BACKSTORY</p>
        <h2>暴雨背后的旧时代</h2>
      </header>
      <ul>
        <li v-for="chapter in worldChapters" :key="chapter.title">
          <h3>{{ chapter.title }}</h3>
          <p>{{ chapter.detail }}</p>
        </li>
      </ul>
    </section>

    <section class="gallery">
      <header>
        <p class="eyebrow">GALLERY</p>
        <h2>雨夜剧照</h2>
      </header>
      <div class="gallery-grid">
        <article v-for="shot in galleryShots" :key="shot.caption">
          <div class="frame"></div>
          <div class="caption">
            <h3>{{ shot.caption }}</h3>
            <p>{{ shot.note }}</p>
          </div>
        </article>
      </div>
    </section>

    <section class="cta">
      <div>
        <p class="eyebrow">预约</p>
        <h2>在个人中心同步「暴雨回响」</h2>
        <p>输入手机号即可接收测试资格，安卓 / iOS 双端同步。</p>
      </div>
      <div class="cta-actions">
        <input placeholder="输入手机号" />
        <button class="btn primary">获取验证码</button>
      </div>
    </section>

    <footer>
      <p>© 2025 Wardrobe Studio · Inspired by Return to Future 1999</p>
      <small>提醒：适度游戏益脑，沉迷游戏伤身。</small>
    </footer>

    <div v-if="showLogin" class="modal-mask">
      <div class="modal">
        <h3>账号密码登录</h3>
        <div class="form">
          <input v-model="loginForm.username" placeholder="用户名" />
          <input v-model="loginForm.password" placeholder="密码" type="password" />
        </div>
        <p v-if="loginError" class="error">{{ loginError }}</p>
        <div class="actions">
          <button class="btn primary" :disabled="loginLoading" @click="doLogin">{{ loginLoading ? '登录中...' : '登录' }}</button>
          <button class="btn ghost" @click="closeLogin">取消</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;600&family=Playfair+Display:wght@500;600&display=swap');

.site {
  display: flex;
  flex-direction: column;
  gap: 3.2rem;
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

.masthead nav {
  display: flex;
  gap: 1.2rem;
  flex-wrap: wrap;
}

.masthead nav a {
  font-size: 0.9rem;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: rgba(223, 233, 240, 0.85);
}

.pill {
  border: 1px solid rgba(214, 184, 116, 0.5);
  background: transparent;
  padding: 0.6rem 1.5rem;
  border-radius: 999px;
  color: inherit;
  cursor: pointer;
}

.hero {
  position: relative;
  padding: 3.5rem;
  border-radius: 36px;
  background: linear-gradient(135deg, rgba(5, 9, 18, 0.95), rgba(15, 36, 48, 0.9));
  border: 1px solid rgba(86, 173, 173, 0.2);
  overflow: hidden;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 2rem;
}

.hero-glow {
  position: absolute;
  inset: 0;
  background: radial-gradient(circle at 18% 30%, rgba(78, 180, 196, 0.55), transparent 50%),
    radial-gradient(circle at 70% 65%, rgba(215, 189, 122, 0.4), transparent 45%);
  mix-blend-mode: screen;
  pointer-events: none;
}

.hero-copy h1 {
  font-family: 'Playfair Display', serif;
  font-size: clamp(2.6rem, 4vw, 3.8rem);
  margin: 1rem 0;
}

.hero-copy p {
  color: rgba(223, 237, 242, 0.82);
  line-height: 1.8;
}

.eyebrow {
  letter-spacing: 0.4em;
  text-transform: uppercase;
  font-size: 0.8rem;
  color: #d9c38f;
}

.hero-actions {
  margin-top: 1.5rem;
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.btn {
  border-radius: 999px;
  padding: 0.85rem 1.8rem;
  font-size: 1rem;
  cursor: pointer;
  border: none;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.btn.primary {
  background: linear-gradient(120deg, #56d0c8, #f0c17a);
  color: #03101a;
  box-shadow: 0 15px 35px rgba(86, 208, 200, 0.35);
}

.btn.ghost {
  border: 1px solid rgba(219, 228, 232, 0.35);
  background: transparent;
  color: white;
}

.btn:hover {
  transform: translateY(-3px);
}

.hero-card {
  position: relative;
  border-radius: 28px;
  padding: 2rem;
  background: rgba(9, 24, 34, 0.65);
  border: 1px solid rgba(86, 173, 173, 0.25);
  backdrop-filter: blur(10px);
  z-index: 1;
}

.tag {
  padding: 0.25rem 0.8rem;
  border-radius: 999px;
  border: 1px solid rgba(214, 184, 116, 0.5);
  font-size: 0.75rem;
  letter-spacing: 0.2em;
  text-transform: uppercase;
}

.hero-stat {
  display: flex;
  gap: 1.5rem;
  margin-top: 1.5rem;
}

.hero-stat span {
  font-size: 0.8rem;
  letter-spacing: 0.2em;
  color: rgba(210, 222, 224, 0.65);
}

.hero-stat strong {
  display: block;
  font-size: 1.6rem;
}

.news header,
.characters header,
.gallery header {
  display: flex;
  align-items: baseline;
  gap: 1rem;
}

.news header a {
  margin-left: auto;
  color: rgba(240, 193, 122, 0.85);
  text-transform: uppercase;
  letter-spacing: 0.2em;
  font-size: 0.75rem;
}

.news-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(230px, 1fr));
  gap: 1rem;
  margin-top: 1.5rem;
}

.news-grid article {
  border-radius: 20px;
  padding: 1.6rem;
  background: rgba(5, 12, 22, 0.8);
  border: 1px solid rgba(86, 173, 173, 0.12);
}

.chip {
  font-size: 0.75rem;
  color: #9bd7cb;
  letter-spacing: 0.4em;
}

.news-grid time {
  font-size: 0.8rem;
  color: rgba(209, 220, 221, 0.6);
}

.media,
.characters {
  border-radius: 28px;
  padding: 2.8rem;
  background: rgba(8, 15, 28, 0.9);
  border: 1px solid rgba(86, 173, 173, 0.1);
}

.rain-layer {
  position: absolute;
  inset: 0;
  overflow: hidden;
  pointer-events: none;
}

.raindrop {
  position: absolute;
  bottom: -20%;
  width: 2px;
  height: 140px;
  background: linear-gradient(0deg, rgba(173, 200, 255, 0), rgba(173, 200, 255, 0.7));
  animation-name: rainRise;
  animation-iteration-count: infinite;
  animation-timing-function: linear;
}

@keyframes rainRise {
  from {
    transform: translateY(0);
    opacity: 0.6;
  }
  to {
    transform: translateY(-180%);
    opacity: 0.2;
  }
}

.media header {
  margin-bottom: 1.5rem;
}

.carousel {
  display: grid;
  grid-template-columns: auto 1fr auto;
  align-items: center;
  gap: 1rem;
}

.carousel-window {
  position: relative;
  overflow: hidden;
  border-radius: 28px;
}

.carousel-slide {
  position: absolute;
  inset: 0;
  opacity: 0;
  transform: scale(0.96);
  transition: opacity 0.6s ease, transform 0.6s ease;
  border-radius: 28px;
  overflow: hidden;
  border: 1px solid rgba(86, 173, 173, 0.12);
}

.carousel-slide.active {
  opacity: 1;
  transform: scale(1);
  position: relative;
}

.carousel-slide video {
  width: 100%;
  height: 320px;
  object-fit: cover;
  display: block;
}

.slide-copy {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 1.5rem;
  background: linear-gradient(180deg, transparent, rgba(5, 5, 10, 0.85));
}

.carousel-btn {
  border: 1px solid rgba(86, 173, 173, 0.5);
  background: transparent;
  color: white;
  border-radius: 999px;
  width: 52px;
  height: 52px;
  font-size: 1.6rem;
  cursor: pointer;
  transition: background 0.3s ease;
}

.carousel-btn:hover {
  background: rgba(255, 255, 255, 0.12);
}

.carousel-dots {
  display: flex;
  justify-content: center;
  gap: 0.6rem;
  margin-top: 1.2rem;
}

.carousel-dots button {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  border: none;
  background: rgba(255, 255, 255, 0.25);
  cursor: pointer;
}

.carousel-dots button.active {
  background: #f0c17a;
}

.character-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
  margin-top: 1.5rem;
}

.character-grid article {
  border-radius: 20px;
  padding: 1.4rem;
  background: linear-gradient(160deg, rgba(17, 42, 53, 0.95), rgba(6, 12, 22, 0.9));
  border: 1px solid rgba(214, 184, 116, 0.14);
}

.badge {
  border: 1px solid rgba(214, 184, 116, 0.4);
  padding: 0.2rem 0.7rem;
  border-radius: 999px;
  font-size: 0.75rem;
  letter-spacing: 0.3em;
}

.role {
  color: rgba(168, 208, 208, 0.8);
  letter-spacing: 0.2em;
  text-transform: uppercase;
  font-size: 0.8rem;
}

.world {
  position: relative;
  border-radius: 32px;
  padding: 3rem;
  border: 1px solid rgba(86, 173, 173, 0.12);
  background: rgba(6, 10, 18, 0.92);
  overflow: hidden;
}

.world-bg {
  position: absolute;
  inset: 0;
  background: radial-gradient(circle at 15% 25%, rgba(92, 194, 197, 0.4), transparent 60%),
    radial-gradient(circle at 80% 20%, rgba(210, 179, 115, 0.3), transparent 55%);
  mix-blend-mode: screen;
  pointer-events: none;
}

.world header {
  position: relative;
  z-index: 1;
}

.world ul {
  position: relative;
  margin-top: 1.5rem;
  list-style: none;
  padding: 0;
  z-index: 1;
  display: grid;
  gap: 1rem;
}

.world li {
  padding: 1.4rem 1.6rem;
  border-radius: 18px;
  border: 1px solid rgba(86, 173, 173, 0.15);
  background: rgba(4, 9, 18, 0.78);
}

.gallery {
  padding: 2rem 0;
}

.gallery-grid {
  margin-top: 1.5rem;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 1rem;
}

.gallery-grid article {
  position: relative;
  border-radius: 24px;
  padding: 1.2rem;
  border: 1px solid rgba(86, 173, 173, 0.12);
  overflow: hidden;
  min-height: 200px;
  background: rgba(5, 14, 24, 0.85);
}

.frame {
  position: absolute;
  inset: 0;
  border: 1px solid rgba(214, 184, 116, 0.2);
  border-radius: 24px;
  background: linear-gradient(140deg, rgba(88, 186, 187, 0.35), transparent);
  opacity: 0.7;
}

.caption {
  position: relative;
  z-index: 1;
}

.cta {
  border-radius: 32px;
  padding: 3rem;
  background: linear-gradient(120deg, rgba(7, 23, 33, 0.96), rgba(27, 50, 61, 0.9));
  border: 1px solid rgba(86, 173, 173, 0.18);
  display: flex;
  flex-wrap: wrap;
  gap: 2rem;
  align-items: center;
}

.cta input {
  border: 1px solid rgba(86, 173, 173, 0.4);
  border-radius: 999px;
  padding: 0.9rem 1.4rem;
  background: transparent;
  color: inherit;
  min-width: 220px;
}

.cta-actions {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

footer {
  text-align: center;
  color: rgba(255, 255, 255, 0.6);
  display: flex;
  flex-direction: column;
  gap: 0.4rem;
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
  width: min(420px, 92vw);
  background: rgba(9, 24, 34, 0.95);
  border: 1px solid rgba(86, 173, 173, 0.25);
  border-radius: 20px;
  padding: 1.6rem;
  color: #ecf3f7;
}

.modal h3 {
  margin: 0 0 1rem 0;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.form input {
  border: 1px solid rgba(86, 173, 173, 0.4);
  border-radius: 12px;
  padding: 0.7rem 1rem;
  background: transparent;
  color: inherit;
}

.actions {
  margin-top: 1rem;
  display: flex;
  gap: 0.8rem;
}

.error {
  color: #f0c17a;
  margin-top: 0.4rem;
}

@media (max-width: 768px) {
  .masthead {
    flex-direction: column;
  }

  .hero {
    padding: 2.2rem;
  }

  .cta {
    padding: 2rem;
  }
}
</style>
