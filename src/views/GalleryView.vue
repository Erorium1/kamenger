<template>
  <div class="gallery-view">
    <section class="section">
      <div class="container">
        <h1 class="section-title animate-fade-in-down">{{ t('gallery.title') }}</h1>
        <p class="gallery-subtitle animate-fade-in-up">{{ t('gallery.subtitle') }}</p>
        <p class="gallery-description animate-fade-in-up" style="margin-bottom: 1.5rem;">{{ t('gallery.description') }}</p>

        <!-- Controls -->
        <div class="controls">
          <div class="chips">
            <button class="chip" :class="{ active: filter === 'all' }" @click="setFilter('all')">{{ t('gallery.filters.all') }}</button>
            <button class="chip" :class="{ active: filter === 'image' }" @click="setFilter('image')">{{ t('gallery.filters.onlyPhotos') }}</button>
            <button class="chip" :class="{ active: filter === 'video' }" @click="setFilter('video')">{{ t('gallery.filters.onlyVideos') }}</button>
          </div>
          <input class="search" type="search" :placeholder="t('gallery.filters.searchPlaceholder')" v-model="query" />
        </div>

        <!-- Creative animated card grid -->
        <div class="masonry" ref="masonryRoot">
          <div
            v-for="(item, index) in filteredItems"
            :key="item.src"
            class="card animate-card"
            :style="{ animationDelay: (index * 60) + 'ms', transform: cardTransforms[index] }"
            @mousemove="onTilt($event, index)"
            @mouseleave="resetTilt(index)"
            @click="openLightbox(indexMap[index])"
            v-intersect
          >
            <div class="card-media-wrapper">
              <img v-if="item.type === 'image'" :src="item.src" :alt="item.name" class="card-media" loading="lazy" />
              <div v-else class="video-badge">▶</div>
            </div>
            <div class="card-overlay">
              <span class="card-caption">{{ briefName(item.name) }}</span>
            </div>
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

// Tilt effect per card
const cardTransforms = ref([])
function onTilt(event, idx) {
  const el = event.currentTarget
  const rect = el.getBoundingClientRect()
  const x = event.clientX - rect.left
  const y = event.clientY - rect.top
  const rx = ((y / rect.height) - 0.5) * -6
  const ry = ((x / rect.width) - 0.5) * 6
  cardTransforms.value[idx] = `perspective(800px) rotateX(${rx}deg) rotateY(${ry}deg)`
}
function resetTilt(idx) {
  cardTransforms.value[idx] = 'perspective(800px) rotateX(0) rotateY(0)'
}

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

// Lightbox state
const lightboxOpen = ref(false)
const lightboxIndex = ref(0)
const currentItem = computed(() => mediaItems.value[lightboxIndex.value])

function openLightbox(originalIndex) {
  lightboxIndex.value = originalIndex
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
  margin-bottom: 1rem;
}

.chips { display: flex; gap: 0.5rem; flex-wrap: wrap; }
.chip {
  border: 1px solid rgba(30,58,138,0.2);
  background: #fff;
  color: var(--kazakh-blue);
  padding: 0.4rem 0.75rem;
  border-radius: 999px;
  cursor: pointer;
  transition: all 200ms ease;
}
.chip:hover { background: rgba(30,58,138,0.06); }
.chip.active { background: var(--kazakh-blue); color: white; border-color: var(--kazakh-blue); }

.search {
  flex: 1;
  min-width: 180px;
  border: 1px solid rgba(30,58,138,0.2);
  border-radius: 10px;
  padding: 0.5rem 0.75rem;
}

/* Masonry-like responsive grid */
.masonry { column-width: 320px; column-gap: 16px; }

.card {
  break-inside: avoid;
  margin-bottom: 16px;
  border-radius: 14px;
  overflow: hidden;
  position: relative;
  background: var(--kazakh-white);
  border: 1px solid rgba(30, 58, 138, 0.1);
  box-shadow: var(--shadow-md);
  cursor: zoom-in;
  transform-origin: center;
  transition: transform 180ms ease;
}

.card-media-wrapper { position: relative; }

.card-media { width: 100%; display: block; transition: transform 400ms ease, filter 400ms ease; }

.video-badge {
  width: 100%; aspect-ratio: 16/9; display: grid; place-items: center;
  background: radial-gradient(circle at 30% 30%, rgba(255,255,255,0.15), rgba(0,0,0,0.5));
  color: #fff; font-size: 2rem;
}

.card-overlay { position: absolute; inset: 0; background: linear-gradient(180deg, rgba(0,0,0,0) 50%, rgba(0,0,0,0.6) 100%); opacity: 0; transition: opacity 300ms ease; display: flex; align-items: flex-end; padding: 0.75rem 1rem; }
.card-caption { color: #fff; font-weight: 500; }

.card:hover .card-media { transform: scale(1.05); filter: saturate(1.1); }
.card:hover .card-overlay { opacity: 1; }

/* Enter animations */
@keyframes cardIn { from { opacity: 0; transform: translateY(16px) scale(0.98); } to { opacity: 1; transform: translateY(0) scale(1); } }
.animate-card { animation: cardIn 600ms cubic-bezier(0.22,1,0.36,1) both; }

/* Lightbox */
.lightbox { position: fixed; inset: 0; background: rgba(0,0,0,0.9); display: flex; align-items: center; justify-content: center; z-index: 2000; }
.lightbox-content { max-width: 95vw; max-height: 92vh; display: flex; flex-direction: column; align-items: center; gap: 0.5rem; }
.lightbox-media { max-width: 95vw; max-height: 86vh; object-fit: contain; border-radius: 10px; box-shadow: var(--shadow-xl); }
.lightbox-caption { color: #fff; opacity: 0.85; }
.lightbox-close { position: absolute; top: 1rem; right: 1rem; }
.lightbox-arrow { position: absolute; top: 50%; transform: translateY(-50%); }
.lightbox-arrow.left { left: 1rem; } .lightbox-arrow.right { right: 1rem; }
</style>
