<template>
  <div class="gallery-view">
    <section class="section">
      <div class="container">
        <h1 class="section-title animate-fade-in-down">{{ t('gallery.title') }}</h1>
        <p class="gallery-subtitle animate-fade-in-up">{{ t('gallery.subtitle') }}</p>
        <p class="gallery-description animate-fade-in-up" style="margin-bottom: 1.5rem;">{{ t('gallery.description') }}
        </p>

        <!-- Controls -->
        <div class="controls">
          <div class="chips">
            <button class="chip" :class="{ active: filter === 'all' }" @click="setFilter('all')">{{
              t('gallery.filters.all') }}</button>
            <button class="chip" :class="{ active: filter === 'image' }" @click="setFilter('image')">{{
              t('gallery.filters.onlyPhotos') }}</button>
            <button class="chip" :class="{ active: filter === 'video' }" @click="setFilter('video')">{{
              t('gallery.filters.onlyVideos') }}</button>
          </div>
          <input class="search" type="search" :placeholder="t('gallery.filters.searchPlaceholder')" v-model="query" />
        </div>

        <!-- 3D Ring Carousel -->
        <div class="ring-carousel">
          <button class="carousel-nav prev btn-secondary" @click="carouselPrev">◀</button>
          <div class="stage">
            <div class="container" ref="containerEl"
              :style="{ '--radius': radius + 'px', '--panel-w': panelW + 'px', '--panel-h': panelH + 'px' }">
              <div class="ring" :style="{ transform: `translateZ(-${radius}px) rotateY(${rotationY}deg)` }">
                <div v-for="(item, index) in filteredItems" :key="item.src" class="panel"
                  :style="{ transform: `rotateY(${angleStep * index}deg) translateZ(${radius}px)` }"
                  @click="openLightbox(indexMap[index])">
                  <template v-if="item.type === 'image'">
                    <div class="media-wrap">
                      <img :src="item.src" :alt="item.name" class="panel-media" loading="lazy"
                        @load="onMediaLoad(item.src)" :class="{ 'is-loading': !isLoaded(item.src) }" />
                      <div v-if="!isLoaded(item.src)" class="loader-overlay"><span class="spinner"></span></div>
                    </div>
                  </template>
                  <template v-else>
                    <div class="video-badge panel-video">▶</div>
                  </template>
                  <div class="panel-caption">{{ briefName(item.name) }}</div>
                </div>
              </div>
            </div>
          </div>
          <button class="carousel-nav next btn-secondary" @click="carouselNext">▶</button>
        </div>
      </div>
    </section>

    <!-- Lightbox fullscreen viewer -->
    <div v-if="lightboxOpen" class="lightbox" @click.self="closeLightbox">
      <button class="lightbox-close btn-secondary" @click="closeLightbox">✕</button>
      <button class="lightbox-arrow left btn-secondary" @click="lightboxPrev">◀</button>
      <div class="lightbox-content">
        <template v-if="currentItem?.type === 'image'">
          <div class="media-wrap">
            <img :src="currentItem.src" :alt="currentItem.name" class="lightbox-media" @load="lightboxLoaded = true" />
            <div v-if="!lightboxLoaded" class="loader-overlay"><span class="spinner"></span></div>
          </div>
        </template>
        <template v-else>
          <video class="lightbox-media" :src="currentItem.src" controls autoplay></video>
        </template>
        <div class="lightbox-caption">{{ displayCaption(currentItem, lightboxIndex) }}</div>
      </div>
      <button class="lightbox-arrow right btn-secondary" @click="lightboxNext">▶</button>
    </div>

    <!-- 3D Model Viewer -->
    <section class="section" style="padding-top: 0;">
      <div class="container">
        <FbxViewer :src="fbxSrc" background="transparent" />
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, inject, computed } from 'vue'
import FbxViewer from '@/components/FbxViewer.vue'
const fbxSrc = 'https://caspianstartup.kz/docs/kemer.fbx'

const translations = inject('translations')
const currentLanguage = inject('currentLanguage')
const t = (key) => {
  const keys = key.split('.')
  let result = translations.value[currentLanguage.value]
  for (const k of keys) result = result?.[k]
  return result || key
}

// Static CDN sources (provided by user)
const cdnSources = [
  'https://res.cloudinary.com/demc61dkq/image/upload/v1758533996/Picture7_jesnga.png',
  'https://res.cloudinary.com/demc61dkq/image/upload/v1758533996/Picture6_a9mh5h.png',
  'https://res.cloudinary.com/demc61dkq/image/upload/v1758533995/Picture5_pagzza.png',
  'https://res.cloudinary.com/demc61dkq/video/upload/v1758533995/WhatsApp_Video_2025-09-19_at_19.26.03_btsbfo.mp4',
  'https://res.cloudinary.com/demc61dkq/image/upload/v1758533993/Picture2_hvkbiu.png',
  'https://res.cloudinary.com/demc61dkq/image/upload/v1758533990/Picture4_ual3yv.png',
  'https://res.cloudinary.com/demc61dkq/image/upload/v1758533988/Picture1_rpcsca.png'
]

const mediaItems = ref([])
const filter = ref('all')
const query = ref('')
const loadedSet = ref(new Set())
const lightboxLoaded = ref(false)

const inferType = (url) => {
  const lower = url.toLowerCase()
  if (lower.endsWith('.mp4') || lower.endsWith('.webm') || lower.endsWith('.ogg')) return 'video'
  return 'image'
}

const buildItems = () => {
  mediaItems.value = cdnSources.map((src) => ({
    type: inferType(src),
    src,
    name: src.split('/').pop()
  }))
}

const filteredItems = computed(() => {
  const q = query.value.trim().toLowerCase()
  return mediaItems.value
    .filter(i => (filter.value === 'all' ? true : i.type === filter.value))
    .filter(i => (!q ? true : i.name.toLowerCase().includes(q)))
})

// Map filtered index to original index for lightbox navigation
const indexMap = computed(() => {
  const map = []
  filteredItems.value.forEach(fi => {
    const idx = mediaItems.value.findIndex(mi => mi.src === fi.src)
    map.push(idx)
  })
  return map
})

// 3D ring carousel state
const rotationY = ref(0)
const containerEl = ref(null)
const radius = ref(520)
const panelW = ref(320)
const panelH = ref(420)
const angleStep = computed(() => (filteredItems.value.length ? 360 / filteredItems.value.length : 0))

function recalcSizes() {
  const el = containerEl.value
  if (!el) return
  const cw = el.clientWidth
  // Panel sized relative to container width
  panelW.value = Math.max(260, Math.min(420, Math.floor(cw * 0.32)))
  panelH.value = Math.floor(panelW.value * 1.25)
  radius.value = Math.max(380, Math.min(900, Math.floor(cw * 0.6)))
}

onMounted(() => {
  buildItems()
  recalcSizes()
  window.addEventListener('resize', recalcSizes)
  window.addEventListener('keydown', onKey)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', recalcSizes)
  window.removeEventListener('keydown', onKey)
})

function carouselNext() { rotationY.value -= angleStep.value }
function carouselPrev() { rotationY.value += angleStep.value }

function setFilter(v) { filter.value = v }

onMounted(() => {
  buildItems()
  window.addEventListener('keydown', onKey)
})

onBeforeUnmount(() => {
  window.removeEventListener('keydown', onKey)
})

function displayCaption(item, index) {
  if (!item) return ''
  const num = (index ?? 0) + 1
  const total = mediaItems.value.length
  const ofWord = t('gallery.slideOf')
  const typeWord = item.type === 'video' ? t('gallery.video') : t('gallery.photos')
  return `${typeWord} • ${num}/${total} ${ofWord}`
}

function briefName(name) {
  if (!name) return ''
  return name.replace(/\.[^/.]+$/, '').slice(0, 30)
}

function onMediaLoad(src) {
  loadedSet.value.add(src)
}
function isLoaded(src) {
  return loadedSet.value.has(src)
}

// Lightbox state
const lightboxOpen = ref(false)
const lightboxIndex = ref(0)
const currentItem = computed(() => mediaItems.value[lightboxIndex.value])

function openLightbox(originalIndex) {
  lightboxIndex.value = originalIndex
  lightboxLoaded.value = false
  lightboxOpen.value = true
}
function closeLightbox() {
  lightboxOpen.value = false
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

// Reveal on scroll directive
const intersect = {
  mounted(el) {
    el.style.opacity = '0'
    el.style.transform += ' translateY(10px)'
    const obs = new IntersectionObserver((entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          el.style.transition = 'opacity 500ms ease, transform 600ms ease'
          el.style.opacity = '1'
          el.style.transform = el.style.transform.replace(' translateY(10px)', '')
          obs.disconnect()
        }
      })
    }, { threshold: 0.1 })
    obs.observe(el)
  }
}

const vIntersect = intersect
</script>

<script>
export default {
  directives: {
    intersect: (typeof vIntersect !== 'undefined' ? vIntersect : undefined)
  }
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
  margin-bottom: 1rem;
}

.controls {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
  margin-bottom: 0;
}

.chips {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.chip {
  border: 1px solid rgba(30, 58, 138, 0.2);
  background: #fff;
  color: var(--kazakh-blue);
  padding: 0.4rem 0.75rem;
  border-radius: 999px;
  cursor: pointer;
  transition: all 200ms ease;
}

.chip:hover {
  background: rgba(30, 58, 138, 0.06);
}

.chip.active {
  background: var(--kazakh-blue);
  color: white;
  border-color: var(--kazakh-blue);
}

.search {
  flex: 1;
  min-width: 180px;
  border: 1px solid rgba(30, 58, 138, 0.2);
  border-radius: 10px;
  padding: 0.5rem 0.75rem;
}

/* 3D ring carousel styles (inspired by referenced CodePen) */
.ring-carousel {
  position: relative;
  display: grid;
  grid-template-columns: 48px 1fr 48px;
  align-items: center;
  gap: 8px;
  width: 100%;
}

.ring-carousel {
  margin-top: 0;
}

.stage {
  margin-top: 0;
}

.carousel-nav {
  height: 44px;
  width: 44px;
  border-radius: 999px;
  display: grid;
  place-items: center;
  z-index: 2;
}

.stage {
  position: relative;
  height: min(72vh, 680px);
  display: grid;
  place-items: center;
  overflow: hidden;
  background: transparent;
  border-radius: 16px;
  width: 100%;
}

.container {
  perspective: 2000px;
  width: 100%;
  height: 100%;
  position: relative;
}

.ring {
  width: 100%;
  height: 30%;
  position: absolute;
  transform-style: preserve-3d;
  transition: transform 800ms cubic-bezier(0.22, 1, 0.36, 1);
}

.panel {
  position: absolute;
  top: 50%;
  left: 50%;
  width: var(--panel-w);
  height: var(--panel-h);
  transform-style: preserve-3d;
  transform-origin: center center;
  border-radius: 14px;
  overflow: hidden;
  box-shadow: var(--shadow-lg);
  background: var(--kazakh-white);
  border: 1px solid rgba(30, 58, 138, 0.1);
  cursor: zoom-in;
}

.panel-media {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.media-wrap {
  position: relative;
}

.loader-overlay {
  position: absolute;
  inset: 0;
  display: grid;
  place-items: center;
  background: linear-gradient(180deg, rgba(0, 0, 0, 0.02), rgba(0, 0, 0, 0.12));
}

.spinner {
  width: 28px;
  height: 28px;
  border: 3px solid rgba(255, 255, 255, 0.6);
  border-top-color: var(--kazakh-blue);
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

.panel-video {
  width: 100%;
  height: 100%;
  display: grid;
  place-items: center;
  color: #fff;
  font-size: 2rem;
  background: radial-gradient(circle at 30% 30%, rgba(255, 255, 255, 0.12), rgba(0, 0, 0, 0.6));
}

.panel-caption {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  color: #fff;
  padding: 0.5rem 0.75rem;
  background: linear-gradient(180deg, rgba(0, 0, 0, 0) 0%, rgba(0, 0, 0, 0.7) 100%);
  font-weight: 500;
}

/* Lightbox */
.lightbox {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.9);
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

.lightbox-arrow.left {
  left: 1rem;
}

.lightbox-arrow.right {
  right: 1rem;
}
</style>
