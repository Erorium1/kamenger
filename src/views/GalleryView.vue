<template>
  <div class="gallery-view">
    <section class="section">
      <div class="container">
        <h1 class="section-title animate-fade-in-down">{{ t('gallery.title') }}</h1>
        <p class="gallery-subtitle animate-fade-in-up">{{ t('gallery.subtitle') }}</p>
        <p class="gallery-description animate-fade-in-up" style="margin-bottom: 1.5rem;">{{ t('gallery.description') }}</p>

        <div class="slider" @mouseenter="pause" @mouseleave="play">
          <div class="slides" :style="slidesStyle">
            <div v-for="(item, index) in mediaItems"
                 :key="item.src"
                 class="slide"
                 :class="{ 'slide-active': index === currentIndex }">
              <template v-if="item.type === 'image'">
                <img :src="item.src" :alt="item.name" class="slide-media" @click="openLightbox(index)" />
              </template>
              <template v-else>
                <video class="slide-media" :src="item.src" controls preload="metadata" @click="openLightbox(index)"></video>
              </template>
              <div class="caption">
                <span>{{ displayCaption(item, index) }}</span>
              </div>
            </div>
          </div>

          <button class="ctrl btn-secondary animate-pulse" @click="prev" aria-label="prev">◀ {{ t('gallery.controls.prev') }}</button>
          <button class="ctrl btn-secondary animate-pulse" @click="next" aria-label="next">{{ t('gallery.controls.next') }} ▶</button>

          <button class="ctrl playpause btn-primary" @click="toggle">
            {{ isPlaying ? t('gallery.controls.pause') : t('gallery.controls.play') }}
          </button>

          <div class="dots">
            <button v-for="(item, i) in mediaItems" :key="i" class="dot" :class="{ active: i === currentIndex }" @click="goTo(i)"></button>
          </div>
        </div>
      </div>
    </section>

    <!-- Lightbox fullscreen viewer -->
    <div v-if="lightboxOpen" class="lightbox" @click.self="closeLightbox">
      <button class="lightbox-close btn-secondary" @click="closeLightbox">✕</button>
      <button class="lightbox-arrow left btn-secondary" @click="lightboxPrev">◀</button>
      <div class="lightbox-content">
        <template v-if="currentItem?.type === 'image'">
          <img :src="currentItem.src" :alt="currentItem.name" class="lightbox-media" />
        </template>
        <template v-else>
          <video class="lightbox-media" :src="currentItem.src" controls autoplay></video>
        </template>
        <div class="lightbox-caption">{{ displayCaption(currentItem, lightboxIndex) }}</div>
      </div>
      <button class="lightbox-arrow right btn-secondary" @click="lightboxNext">▶</button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, inject, computed } from 'vue'

const translations = inject('translations')
const currentLanguage = inject('currentLanguage')
const t = (key) => {
  const keys = key.split('.')
  let result = translations.value[currentLanguage.value]
  for (const k of keys) result = result?.[k]
  return result || key
}

// Import media from assets/img via Vite glob
const imageModules = import.meta.glob('../assets/img/*.{png,jpg,jpeg,gif,webp}', { eager: true, as: 'url' })
const videoModules = import.meta.glob('../assets/img/*.{mp4,webm,ogg}', { eager: true, as: 'url' })

const mediaItems = ref([])
const currentIndex = ref(0)
const intervalMs = 4000
let timer = null
const isPlaying = ref(true)

const buildItems = () => {
  const images = Object.entries(imageModules).map(([key, url]) => ({
    type: 'image',
    src: url,
    name: key.split('/').pop()
  }))
  const videos = Object.entries(videoModules).map(([key, url]) => ({
    type: 'video',
    src: url,
    name: key.split('/').pop()
  }))
  mediaItems.value = [...images, ...videos]
}

const next = () => {
  if (mediaItems.value.length === 0) return
  currentIndex.value = (currentIndex.value + 1) % mediaItems.value.length
}
const prev = () => {
  if (mediaItems.value.length === 0) return
  currentIndex.value = (currentIndex.value - 1 + mediaItems.value.length) % mediaItems.value.length
}
const goTo = (i) => {
  if (mediaItems.value.length === 0) return
  currentIndex.value = i
}

const play = () => {
  isPlaying.value = true
  startTimer()
}
const pause = () => {
  isPlaying.value = false
  stopTimer()
}
const toggle = () => {
  isPlaying.value ? pause() : play()
}

const startTimer = () => {
  stopTimer()
  if (mediaItems.value.length === 0) return
  timer = setInterval(next, intervalMs)
}
const stopTimer = () => {
  if (timer) clearInterval(timer)
  timer = null
}

onMounted(() => {
  buildItems()
  startTimer()
  window.addEventListener('keydown', onKey)
})

onBeforeUnmount(() => {
  stopTimer()
  window.removeEventListener('keydown', onKey)
})

const slidesStyle = computed(() => ({
  transform: `translateX(-${currentIndex.value * 100}%)`
}))

function displayCaption(item, index) {
  if (!item) return ''
  const num = (index ?? 0) + 1
  const total = mediaItems.value.length
  const ofWord = t('gallery.slideOf')
  const typeWord = item.type === 'video' ? t('gallery.video') : t('gallery.photos')
  return `${typeWord} • ${num}/${total} ${ofWord}`
}

// Lightbox state
const lightboxOpen = ref(false)
const lightboxIndex = ref(0)
const currentItem = computed(() => mediaItems.value[lightboxIndex.value])

function openLightbox(index) {
  lightboxIndex.value = index
  lightboxOpen.value = true
  pause()
}
function closeLightbox() {
  lightboxOpen.value = false
  play()
}
function lightboxNext() {
  if (!mediaItems.value.length) return
  lightboxIndex.value = (lightboxIndex.value + 1) % mediaItems.value.length
}
function lightboxPrev() {
  if (!mediaItems.value.length) return
  lightboxIndex.value = (lightboxIndex.value - 1 + mediaItems.value.length) % mediaItems.value.length
}
function onKey(e) {
  if (!lightboxOpen.value) return
  if (e.key === 'Escape') closeLightbox()
  if (e.key === 'ArrowRight') lightboxNext()
  if (e.key === 'ArrowLeft') lightboxPrev()
}
</script>

<style scoped>
.gallery-subtitle {
  text-align: center;
  color: var(--kazakh-blue);
  font-weight: 600;
  margin-bottom: 0.5rem;
}
.gallery-description {
  text-align: center;
  color: #475569;
}

.slider {
  position: relative;
  overflow: hidden;
  border-radius: 14px;
  background: var(--kazakh-white);
  border: 1px solid rgba(30, 58, 138, 0.1);
  box-shadow: var(--shadow-lg);
}

.slides {
  display: flex;
  transition: transform 600ms cubic-bezier(0.22, 1, 0.36, 1);
}

.slide {
  min-width: 100%;
  position: relative;
}

.slide-media {
  width: 100%;
  height: 58vh;
  object-fit: cover;
  display: block;
  cursor: zoom-in;
}

.caption {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(180deg, rgba(0,0,0,0) 0%, rgba(0,0,0,0.6) 100%);
  color: white;
  padding: 0.75rem 1rem;
  font-size: 0.95rem;
}

.ctrl {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  z-index: 5;
  padding: 0.5rem 0.75rem;
}

.ctrl:first-of-type { left: 0.75rem; }
.ctrl:nth-of-type(2) { right: 0.75rem; }
.playpause {
  top: 0.75rem;
  right: 0.75rem;
  transform: none;
}

.dots {
  position: absolute;
  left: 50%;
  transform: translateX(-50%);
  bottom: 0.75rem;
  display: flex;
  gap: 8px;
}

.dot {
  width: 10px;
  height: 10px;
  border-radius: 999px;
  background: rgba(255,255,255,0.6);
  border: 1px solid rgba(255,255,255,0.9);
}

.dot.active { background: var(--kazakh-gold); }

/* Lightbox */
.lightbox {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.9);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
}

.lightbox-content {
  max-width: 95vw;
  max-height: 92vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

.lightbox-media {
  max-width: 95vw;
  max-height: 86vh;
  object-fit: contain;
  border-radius: 10px;
  box-shadow: var(--shadow-xl);
}

.lightbox-caption {
  color: #fff;
  opacity: 0.85;
}

.lightbox-close {
  position: absolute;
  top: 1rem;
  right: 1rem;
}

.lightbox-arrow {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
}

.lightbox-arrow.left { left: 1rem; }
.lightbox-arrow.right { right: 1rem; }

@media (max-width: 768px) {
  .slide-media { height: 50vh; }
}

@media (max-width: 480px) {
  .slide-media { height: 42vh; }
}
</style>
