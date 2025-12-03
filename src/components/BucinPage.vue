<template>
  <div class="page">
    <!-- Navbar -->
    <nav class="navbar" :class="{ 'scrolled': isScrolled }">
      <div class="brand">
        <span class="heart-icon">💖</span> KitaSelamanya
      </div>
      <ul class="nav-links">
        <li><button class="btn-ghost" @click="scrollToSection('gallery')">Galeri</button></li>
        <li><button class="btn-ghost" @click="scrollToSection('videos')">Video</button></li>
        <li><button class="btn-ghost" @click="scrollToSection('story')">Cerita</button></li>
      </ul>
    </nav>

    <!-- Hero -->
    <header class="hero">
      <div class="hero-bg"></div>
      <div class="hero-content">
        <h1 class="title-animate">Our Love Story</h1>
        <p class="subtitle-animate">Dunia terasa lebih indah saat ada kamu 🌸</p>
        <button class="btn-primary glow-effect" @click="scrollToSection('gallery')">
          Lihat Kenangan Kita
        </button>
      </div>
      <div class="scroll-indicator">
        <span>💕</span>
      </div>
    </header>

    <!-- Music Player -->
    <div class="music-player" :class="{ 'playing': isPlaying }">
      <div class="disk-wrapper">
        <div class="disk" :style="{ animationPlayState: isPlaying ? 'running' : 'paused' }">
          <div class="disk-center"></div>
        </div>
      </div>
      <div class="player-info">
        <span class="song-title">About You</span>
        <span class="artist">The 1975</span>
      </div>
      <button class="play-btn" @click="toggleMusic">
        {{ isPlaying ? '⏸' : '▶' }}
      </button>
      <audio ref="audioPlayer" loop>
        <source :src="bgMusic" type="audio/mp3">
      </audio>
    </div>

    <!-- Galeri Foto -->
    <section ref="gallerySection" class="section gallery-section">
      <div class="section-header">
        <h2 class="section-title">Momen Manis</h2>
        <div class="divider"></div>
      </div>
      
      <div class="gallery-grid">
        <div
          v-for="(p, i) in photos"
          :key="i"
          class="photo-card"
          @click="openLightbox(i, 'photos')"
        >
          <div class="img-wrapper">
            <img :src="p.src" :alt="p.title" loading="lazy" />
          </div>
          <div class="photo-overlay">
            <span class="photo-date">{{ p.date }}</span>
            <h3 class="photo-title">{{ p.title }}</h3>
          </div>
        </div>
      </div>
    </section>

    <!-- Video Section -->
    <section ref="videoSection" class="section video-section">
      <div class="section-header">
        <h2 class="section-title">Video Kita</h2>
        <div class="divider"></div>
      </div>
      
      <div class="video-grid">
        <div v-for="(v, i) in videos" :key="i" class="video-card-simple">
          <video controls preload="metadata">
            <source :src="v.src" type="video/mp4">
            Browser kamu tidak support video.
          </video>
        </div>
      </div>
    </section>

    <!-- Trending / Highlights -->
    <section class="section highlight-section">
      <div class="section-header">
        <h2 class="section-title">Highlight</h2>
        <div class="divider"></div>
      </div>
      <div class="highlight-carousel">
        <div v-for="(trend, i) in trending" :key="i" class="highlight-card" @click="openLightbox(i, 'trending')">
          <img :src="trend.src" :alt="trend.title" />
          <div class="highlight-content">
            <h3>{{ trend.title }}</h3>
            <p>{{ trend.desc }}</p>
            <div class="stats">
              <span>❤️ {{ trend.likes }}</span>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Cerita -->
    <section ref="storySection" class="section story-section">
      <div class="story-container">
        <h2 class="section-title">Cerita Kita</h2>
        <p class="story-text">
          "Terima kasih sudah mewarnai hari-hariku dengan warna pink yang indah. 
          Kamu adalah alasan kenapa aku selalu tersenyum setiap hari. I Love You! 💖"
        </p>
      </div>
    </section>

    <!-- Footer -->
    <footer class="footer">
      <p>Dibuat dengan penuh Cinta 💖 untukmu By Andikaa.</p>
    </footer>

    <!-- Lightbox -->
    <Transition name="fade">
      <div v-if="lightboxOpen" class="lightbox" @click.self="closeLightbox">
        <button class="close-btn" @click="closeLightbox">✕</button>
        <div class="lightbox-content">
          <button class="nav-btn prev" @click.stop="prevImage">‹</button>
          <div class="lightbox-media">
            <img :src="activeImage.src" :alt="activeImage.title" />
            <div class="lightbox-details">
              <h3>{{ activeImage.title }}</h3>
              <p>{{ activeImage.desc }}</p>
            </div>
          </div>
          <button class="nav-btn next" @click.stop="nextImage">›</button>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

// Assets Import
import bgMusicFile from '../assets/The 1975 - About You (Official) - The1975VEVO.mp3'

// Photos
import foto1 from '../assets/foto1.jpg'
import foto2 from '../assets/foto2.jpg'
import foto3 from '../assets/foto3.jpg'
import foto4 from '../assets/foto4.jpg'
import foto5 from '../assets/foto5.jpg'
import foto6 from '../assets/foto6.jpg'
import kita1 from '../assets/kita1.jpg'
import kita2 from '../assets/kita2.jpg'
import kita3 from '../assets/kita3.jpg'
import kita4 from '../assets/kita4.jpg'
import kita5 from '../assets/kita5.jpg'
import kita6 from '../assets/kita6.jpg'
import kita7 from '../assets/kita7.jpg'
import kita13 from '../assets/kita13.jpg'

// Videos
import video8 from '../assets/kita8.mp4'
import video9 from '../assets/kita9.mp4'
import video10 from '../assets/kita10.mp4'
import video12 from '../assets/kita12.mp4'
import video14 from '../assets/kita14.mp4'

const bgMusic = bgMusicFile

// Data
const photos = ref([
  { src: kita13, title: 'Spesial', date: 'Momen Kita', desc: 'Senyummu mengalihkan duniaku.' },
  { src: kita1, title: 'Awal Cerita', date: 'Day 1', desc: 'Pertemuan yang tak terlupakan.' },
  { src: kita2, title: 'Jalan-jalan', date: 'Weekend', desc: 'Menikmati hari minggu bersamamu.' },
  { src: kita3, title: 'Sweet Moment', date: 'Date Night', desc: 'Makan malam romantis.' },
  { src: kita4, title: 'Candid', date: 'Random', desc: 'Kamu cantik apa adanya.' },
  { src: kita5, title: 'Traveling', date: 'Holiday', desc: 'Petualangan baru dimulai.' },
  { src: kita6, title: 'Sunset', date: 'Sore Itu', desc: 'Menunggu matahari terbenam.' },
  { src: kita7, title: 'Bahagia', date: 'Forever', desc: 'Semoga selamanya seperti ini.' },
  { src: foto1, title: 'Estetik', date: 'Vibes', desc: 'Foto ala-ala pinterest.' },
  { src: foto2, title: 'Warmth', date: 'Cozy', desc: 'Kehangatan di tengah hujan.' },
  { src: foto3, title: 'Laugh', date: 'Happy', desc: 'Tawamu canduku.' },
  { src: foto4, title: 'Together', date: 'Us', desc: 'Kita lawan dunia.' },
  { src: foto5, title: 'Dream', date: 'Future', desc: 'Mimpi-mimpi kita.' },
  { src: foto6, title: 'Memory', date: 'Throwback', desc: 'Mengingat masa lalu.' },
])

const videos = ref([
  { src: video8, title: 'Vlog Mini', desc: 'Keseruan kita hari ini!' },
  { src: video9, title: 'Tiktok Bareng', desc: 'Iseng-iseng berhadiah.' },
  { src: video10, title: 'Trip Singkat', desc: 'Jalan-jalan dadakan.' },
  { src: video12, title: 'Anniversary', desc: 'Merayakan hari jadi kita.' },
  { src: video14, title: 'Anniversary', desc: 'Hal Random.' },
])

const trending = ref([
  { src: kita13, title: 'Best Photo', desc: 'Paling favorit!', likes: 9999 },
  { src: kita2, title: 'Cute', desc: 'Gemes banget.', likes: 8888 },
  { src: kita7, title: 'Goals', desc: 'Couple goals katanya.', likes: 7777 },
])

// State
const isScrolled = ref(false)
const isPlaying = ref(false)
const audioPlayer = ref(null)
const lightboxOpen = ref(false)
const currentIdx = ref(0)
const activeSource = ref('photos') // 'photos' | 'trending'

// Scroll Handler
const handleScroll = () => {
  isScrolled.value = window.scrollY > 50
}

onMounted(() => {
  window.addEventListener('scroll', handleScroll)
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
})

// Music Control
const toggleMusic = () => {
  if (!audioPlayer.value) return
  if (isPlaying.value) {
    audioPlayer.value.pause()
  } else {
    audioPlayer.value.play().catch(e => console.log('Autoplay prevented', e))
  }
  isPlaying.value = !isPlaying.value
}

// Navigation
const gallerySection = ref(null)
const videoSection = ref(null)
const storySection = ref(null)

const scrollToSection = (section) => {
  const el = {
    gallery: gallerySection,
    videos: videoSection,
    story: storySection
  }[section]
  
  el.value?.scrollIntoView({ behavior: 'smooth', block: 'start' })
}

// Lightbox
const activeList = computed(() => activeSource.value === 'photos' ? photos.value : trending.value)
const activeImage = computed(() => activeList.value[currentIdx.value] || {})

const openLightbox = (index, source) => {
  activeSource.value = source
  currentIdx.value = index
  lightboxOpen.value = true
  document.body.style.overflow = 'hidden'
}

const closeLightbox = () => {
  lightboxOpen.value = false
  document.body.style.overflow = ''
}

const nextImage = () => {
  currentIdx.value = (currentIdx.value + 1) % activeList.value.length
}

const prevImage = () => {
  currentIdx.value = (currentIdx.value - 1 + activeList.value.length) % activeList.value.length
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Dancing+Script:wght@600;700&family=Outfit:wght@300;400;600&display=swap');

:root {
  --primary: #ff85a2; /* Cute Pink */
  --secondary: #ffb7ce; /* Soft Pink */
  --accent: #fff0f5; /* Lavender Blush */
  --text: #5e2a40; /* Dark Pink/Brown for text */
  --glass: rgba(255, 255, 255, 0.25);
  --glass-border: rgba(255, 255, 255, 0.4);
  --dark-glass: rgba(0, 0, 0, 0.1);
}

.page {
  /* Pink Lucu Gradient */
  background: linear-gradient(180deg, #ffdde1 0%, #ee9ca7 100%);
  color: var(--text);
  font-family: 'Outfit', sans-serif;
  min-height: 100vh;
  overflow-x: hidden;
}

/* Navbar */
.navbar {
  position: fixed;
  top: 0;
  width: 100%;
  padding: 20px 40px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  z-index: 1000;
  transition: 0.3s ease;
}
.navbar.scrolled {
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(10px);
  padding: 15px 40px;
  box-shadow: 0 4px 20px rgba(255, 133, 162, 0.2);
}
.brand {
  font-family: 'Dancing Script', cursive;
  font-size: 28px;
  font-weight: 700;
  color: var(--primary);
  display: flex;
  align-items: center;
  gap: 8px;
  text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
}
.heart-icon {
  animation: beat 1.5s infinite;
}
@keyframes beat {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.2); }
}
.nav-links {
  display: flex;
  gap: 20px;
  list-style: none;
}
.btn-ghost {
  background: transparent;
  border: none;
  color: var(--text);
  font-size: 16px;
  cursor: pointer;
  transition: 0.3s;
  padding: 8px 16px;
  border-radius: 20px;
  font-weight: 600;
}
.btn-ghost:hover {
  color: var(--primary);
  background: rgba(255, 255, 255, 0.5);
}

/* Hero */
.hero {
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  position: relative;
  overflow: hidden;
}
.hero-bg {
  position: absolute;
  inset: 0;
  background: url('../assets/foto1.jpg') center/cover no-repeat;
  filter: brightness(0.6) blur(3px);
  transform: scale(1.1);
  animation: zoomBg 20s infinite alternate;
}
@keyframes zoomBg {
  from { transform: scale(1.1); }
  to { transform: scale(1.2); }
}
.hero-content {
  position: relative;
  z-index: 2;
  max-width: 800px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(5px);
  border-radius: 30px;
  border: 1px solid var(--glass-border);
  box-shadow: 0 10px 40px rgba(0,0,0,0.1);
}
.title-animate {
  font-family: 'Dancing Script', cursive;
  font-size: 5rem;
  color: #fff;
  margin-bottom: 20px;
  text-shadow: 0 4px 15px rgba(255, 77, 136, 0.6);
  animation: fadeInDown 1.2s ease-out;
}
.subtitle-animate {
  font-size: 1.4rem;
  margin-bottom: 40px;
  color: #fff;
  font-weight: 600;
  text-shadow: 0 2px 5px rgba(0,0,0,0.2);
  animation: fadeInUp 1.2s ease-out 0.3s backwards;
}
.btn-primary {
  background: linear-gradient(45deg, var(--primary), #ff9a9e);
  border: none;
  padding: 14px 40px;
  border-radius: 50px;
  color: white;
  font-weight: 600;
  font-size: 18px;
  cursor: pointer;
  transition: 0.3s;
  box-shadow: 0 5px 15px rgba(255, 133, 162, 0.4);
  animation: fadeInUp 1.2s ease-out 0.6s backwards;
}
.btn-primary:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 30px rgba(255, 133, 162, 0.6);
}
.scroll-indicator {
  position: absolute;
  bottom: 30px;
  animation: bounce 2s infinite;
  font-size: 2rem;
  opacity: 0.8;
}
@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-10px); }
  60% { transform: translateY(-5px); }
}

/* Music Player */
.music-player {
  position: fixed;
  bottom: 30px;
  right: 30px;
  background: rgba(255, 255, 255, 0.85);
  backdrop-filter: blur(10px);
  padding: 12px 20px;
  border-radius: 50px;
  display: flex;
  align-items: center;
  gap: 15px;
  box-shadow: 0 8px 30px rgba(0,0,0,0.1);
  z-index: 999;
  border: 1px solid var(--glass-border);
  transition: 0.3s;
}
.music-player:hover {
  transform: scale(1.05);
}
.disk-wrapper {
  width: 40px;
  height: 40px;
}
.disk {
  width: 100%;
  height: 100%;
  background: linear-gradient(45deg, #333, #000);
  border-radius: 50%;
  position: relative;
  animation: spin 3s linear infinite;
  animation-play-state: paused;
  border: 2px solid var(--primary);
}
.disk-center {
  position: absolute;
  inset: 12px;
  background: var(--primary);
  border-radius: 50%;
}
@keyframes spin {
  to { transform: rotate(360deg); }
}
.player-info {
  display: flex;
  flex-direction: column;
  font-size: 12px;
}
.song-title {
  font-weight: 700;
  color: var(--primary);
}
.artist {
  color: var(--text);
}
.play-btn {
  background: var(--primary);
  border: none;
  color: white;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: 0.2s;
  box-shadow: 0 4px 10px rgba(255, 133, 162, 0.3);
}
.play-btn:hover {
  transform: scale(1.1);
}

/* Sections */
.section {
  padding: 80px 20px;
  max-width: 1200px;
  margin: 0 auto;
}
.section-header {
  text-align: center;
  margin-bottom: 60px;
}
.section-title {
  font-family: 'Dancing Script', cursive;
  font-size: 3.5rem;
  color: var(--primary);
  margin-bottom: 10px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.05);
}
.divider {
  width: 80px;
  height: 5px;
  background: var(--primary);
  margin: 0 auto;
  border-radius: 10px;
}

/* Gallery Grid */
.gallery-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 25px;
}
.photo-card {
  position: relative;
  border-radius: 20px;
  overflow: hidden;
  cursor: pointer;
  height: 350px;
  transition: 0.4s ease;
  box-shadow: 0 10px 20px rgba(0,0,0,0.05);
  background: #fff;
}
.photo-card:hover {
  transform: translateY(-10px);
  box-shadow: 0 15px 40px rgba(255, 133, 162, 0.2);
}
.img-wrapper {
  width: 100%;
  height: 100%;
}
.img-wrapper img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: 0.5s;
}
.photo-card:hover img {
  transform: scale(1.1);
}
.photo-overlay {
  position: absolute;
  inset: 0;
  background: linear-gradient(to top, rgba(255, 77, 136, 0.8), transparent);
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  padding: 20px;
  opacity: 0;
  transition: 0.3s;
}
.photo-card:hover .photo-overlay {
  opacity: 1;
}
.photo-title {
  font-size: 1.4rem;
  font-weight: 700;
  color: #fff;
  font-family: 'Dancing Script', cursive;
}
.photo-date {
  font-size: 0.9rem;
  color: #fff;
  margin-bottom: 5px;
  font-weight: 600;
}

/* Video Section - Simple */
.video-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 30px;
}
.video-card-simple {
  background: #fff;
  border-radius: 20px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(0,0,0,0.1);
  padding: 10px;
}
.video-card-simple video {
  width: 100%;
  height: 250px;
  object-fit: cover;
  border-radius: 15px;
  background: #000;
}

/* Highlight Carousel */
.highlight-carousel {
  display: flex;
  gap: 25px;
  overflow-x: auto;
  padding-bottom: 20px;
  scroll-snap-type: x mandatory;
}
.highlight-carousel::-webkit-scrollbar {
  height: 8px;
}
.highlight-carousel::-webkit-scrollbar-thumb {
  background: var(--primary);
  border-radius: 4px;
}
.highlight-card {
  min-width: 280px;
  height: 400px;
  border-radius: 20px;
  overflow: hidden;
  position: relative;
  scroll-snap-align: center;
  cursor: pointer;
  box-shadow: 0 10px 25px rgba(0,0,0,0.1);
}
.highlight-card img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.highlight-content {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 20px;
  background: linear-gradient(to top, rgba(0,0,0,0.7), transparent);
  color: #fff;
}

/* Story */
.story-container {
  background: rgba(255, 255, 255, 0.6);
  backdrop-filter: blur(10px);
  padding: 60px;
  border-radius: 30px;
  text-align: center;
  border: 2px solid #fff;
  position: relative;
  overflow: hidden;
  box-shadow: 0 10px 40px rgba(255, 133, 162, 0.15);
}
.story-container::before {
  content: '"';
  font-family: 'Dancing Script', cursive;
  font-size: 10rem;
  position: absolute;
  top: -20px;
  left: 20px;
  color: rgba(255, 133, 162, 0.2);
}
.story-text {
  font-size: 1.6rem;
  line-height: 1.8;
  font-style: italic;
  max-width: 800px;
  margin: 0 auto;
  color: var(--text);
}

/* Footer */
.footer {
  text-align: center;
  padding: 40px;
  opacity: 0.8;
  font-size: 1rem;
  font-weight: 600;
  color: var(--text);
}

/* Lightbox */
.lightbox {
  position: fixed;
  inset: 0;
  background: rgba(255, 255, 255, 0.95);
  z-index: 2000;
  display: flex;
  align-items: center;
  justify-content: center;
}
.lightbox-content {
  display: flex;
  align-items: center;
  gap: 20px;
  width: 100%;
  max-width: 1200px;
  padding: 20px;
}
.lightbox-media {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}
.lightbox-media img {
  max-height: 80vh;
  max-width: 100%;
  border-radius: 20px;
  box-shadow: 0 20px 50px rgba(255, 133, 162, 0.3);
}
.lightbox-details {
  text-align: center;
}
.lightbox-details h3 {
  color: var(--primary);
  font-size: 2rem;
  margin-bottom: 5px;
  font-family: 'Dancing Script', cursive;
}
.lightbox-details p {
  color: var(--text);
}
.nav-btn {
  background: rgba(255, 133, 162, 0.2);
  border: none;
  color: var(--primary);
  font-size: 2rem;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  cursor: pointer;
  transition: 0.3s;
}
.nav-btn:hover {
  background: var(--primary);
  color: #fff;
}
.close-btn {
  position: absolute;
  top: 30px;
  right: 30px;
  background: transparent;
  border: none;
  color: var(--text);
  font-size: 2rem;
  cursor: pointer;
  transition: 0.3s;
}
.close-btn:hover {
  color: var(--primary);
  transform: rotate(90deg);
}

/* Animations */
@keyframes fadeInDown {
  from { opacity: 0; transform: translateY(-30px); }
  to { opacity: 1; transform: translateY(0); }
}
@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(30px); }
  to { opacity: 1; transform: translateY(0); }
}

/* Responsive */
@media (max-width: 768px) {
  .title-animate { font-size: 3rem; }
  .navbar { padding: 15px 20px; }
  .nav-links { display: none; }
  .music-player { bottom: 20px; right: 20px; padding: 10px 15px; }
  .disk-wrapper { display: none; }
  .story-text { font-size: 1.2rem; }
}
</style>
