<template>
  <div class="home">
    <!-- Героический баннер -->
    <section class="hero-banner">
      <div class="hero-background">
        <div class="hero-overlay"></div>
      </div>

      <div class="hero-content-wrapper">
        <div class="hero-content">
          <div class="hero-text animate-fade-in-up">
            <h1 class="hero-title animate-bounce-in">
              {{ t('home.title') }}
            </h1>
            <h2 class="hero-subtitle animate-flip-in-x" style="animation-delay: 0.3s;">
              {{ t('home.subtitle') }}
            </h2>
            <p class="hero-description animate-fade-in-up" style="animation-delay: 0.6s;">
              {{ t('home.description') }}
            </p>
            <blockquote class="hero-quote animate-shimmer" style="animation-delay: 0.9s;">
              "{{ t('home.heroQuote') }}"
            </blockquote>
            <div class="hero-actions animate-fade-in-up" style="animation-delay: 1.2s;">
              <router-link to="/biography" class="btn btn-primary animate-glow">
                {{ t('home.learnMore') }}
              </router-link>
              <button @click="scrollToTimeline" class="btn btn-secondary animate-pulse">
                Хронология
              </button>
            </div>
          </div>

          <div class="hero-image animate-fade-in-up" style="animation-delay: 0.3s">
            <div class="hero-portrait animate-float">
              <div class="portrait-placeholder">
                <!-- <span class="portrait-icon animate-rotate-3d">🏅</span>
                <p class="portrait-text">{{ t('home.portrait') }}</p> -->
                <img
                  src="https://res.cloudinary.com/demc61dkq/image/upload/v1758704604/WhatsApp_Image_2025-09-24_at_11.02.07_km0ks3.jpg"
                  alt="" style="transform: scale(0.39);">
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>

    <!-- Canvas интеграция -->
    <section class="canvas-section section">
      <div class="container">
        <h2 class="section-title">{{ t('home.timeline') }}</h2>
        <div class="canvas-container">
          <div class="canvas-placeholder">
            <div class="canvas-icon">📊</div>
            <h3>{{ t('home.canvasTitle') }}</h3>
            <p>{{ t('home.canvasDescription') }}</p>
            <a href="https://www.canva.com/design/DAGS31Y2ZwE/_vvTNG9fnZMrCN5h13PvOg/edit?utm_content=DAGS31Y2ZwE&utm_campaign=designshare&utm_medium=link2&utm_source=sharebutton"
              target="_blank" rel="noopener" class="btn btn-primary">
              {{ t('home.viewCanvas') }}
            </a>
          </div>
        </div>
      </div>
    </section>

    <!-- Краткая биография -->
    <section class="bio-preview section">
      <div class="container">
        <div class="bio-content">
          <div class="bio-text animate-slide-in-left">
            <h2 class="section-title">{{ t('biography.title') }}</h2>
            <div class="bio-summary">
              <p class="bio-paragraph">
                {{ t('biography.summary1') }}
              </p>
              <p class="bio-paragraph">
                {{ t('biography.summary2') }}
              </p>
              <p class="bio-paragraph">
                {{ t('biography.summary3') }}
              </p>
            </div>
            <router-link to="/biography" class="btn btn-primary">
              {{ t('biography.readFull') }}
            </router-link>
          </div>

          <div class="bio-stats animate-slide-in-right">
            <div class="stat-card animate-bounce-in" style="animation-delay: 0.1s;">
              <div class="stat-number">1925</div>
              <div class="stat-label">{{ t('home.birthYear') }}</div>
            </div>
            <div class="stat-card animate-bounce-in" style="animation-delay: 0.2s;">
              <div class="stat-number">1943</div>
              <div class="stat-label">{{ t('home.enlistYear') }}</div>
            </div>
            <div class="stat-card animate-bounce-in" style="animation-delay: 0.3s;">
              <div class="stat-number">3</div>
              <div class="stat-label">{{ t('home.orders') }}</div>
            </div>
            <div class="stat-card animate-bounce-in animate-glow" style="animation-delay: 0.4s;">
              <div class="stat-number">100%</div>
              <div class="stat-label">{{ t('home.hero') }}</div>
            </div>
          </div>
        </div>
      </div>
    </section>

  </div>
</template>

<script setup>
import { inject } from 'vue'

// Инъекция переводов и текущего языка
const translations = inject('translations')
const currentLanguage = inject('currentLanguage')

// Функция перевода
const t = (key) => {
  const keys = key.split('.')
  let result = translations.value[currentLanguage.value]

  for (const k of keys) {
    result = result?.[k]
  }

  return result || key
}

// Прокрутка к таймлайну
const scrollToTimeline = () => {
  const timeline = document.querySelector('.canvas-section')
  if (timeline) {
    timeline.scrollIntoView({ behavior: 'smooth' })
  }
}
</script>

<style scoped>
.home {
  min-height: 100vh;
}

/* Героический баннер */
.hero-banner {
  position: relative;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  width: 100vw;
  margin-left: calc(-50vw + 50%);
}

.hero-background {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(135deg, #1E3A8A 0%, #D4A017 100%);
  z-index: 1;
}

.hero-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.3);
  z-index: 2;
}

.hero-content-wrapper {
  position: relative;
  z-index: 3;
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.hero-content {
  display: flex;
  align-items: center;
  gap: 60px;
  width: 100%;
  max-width: 1100px;
}

.hero-text {
  color: var(--kazakh-white);
  flex: 1;
  min-width: 0;
}

.hero-image {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: center;
}

.hero-title {
  font-family: 'Merriweather', serif;
  font-size: 3.5rem;
  font-weight: 700;
  margin-bottom: 1rem;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
  line-height: 1.2;
}

.hero-subtitle {
  font-size: 1.5rem;
  font-weight: 400;
  margin-bottom: 1.5rem;
  color: var(--kazakh-gold);
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
}

.hero-description {
  font-size: 1.2rem;
  line-height: 1.6;
  margin-bottom: 2rem;
  opacity: 0.95;
}

.hero-quote {
  font-size: 1.1rem;
  font-style: italic;
  margin-bottom: 2.5rem;
  padding: 1.5rem;
  background: rgba(255, 255, 255, 0.1);
  border-left: 4px solid var(--kazakh-gold);
  border-radius: 8px;
  backdrop-filter: blur(10px);
}

.hero-actions {
  display: flex;
  gap: 1rem;
  flex-wrap: wrap;
}

.hero-image {
  display: flex;
  justify-content: center;
  align-items: center;
}

.hero-portrait {
  width: 300px;
  height: 400px;
  border-radius: 20px;
  overflow: hidden;
  box-shadow:
    var(--shadow-xl),
    0 0 30px rgba(212, 160, 23, 0.3),
    inset 0 0 50px rgba(255, 255, 255, 0.1);
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(15px);
  border: 2px solid rgba(255, 255, 255, 0.2);
  transform-style: preserve-3d;
  transition: all 0.5s ease;
  margin-top: 12px;
}

.hero-portrait:hover {
  transform: translateY(-10px) rotateX(5deg) rotateY(5deg);
  box-shadow:
    0 20px 40px rgba(0, 0, 0, 0.3),
    0 0 50px rgba(212, 160, 23, 0.5),
    inset 0 0 60px rgba(255, 255, 255, 0.2);
}

.portrait-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  color: var(--kazakh-white);
}

.portrait-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
  filter: drop-shadow(0 0 10px rgba(255, 215, 0, 0.8));
  transform-style: preserve-3d;
}

.portrait-text {
  font-size: 1.1rem;
  opacity: 0.9;
}

/* Canvas секция */
.canvas-section {
  background: var(--kazakh-gray);
}

.canvas-container {
  max-width: 1000px;
  margin: 0 auto;
  padding: 0 2rem;
}

.canvas-placeholder {
  background: var(--kazakh-white);
  border: 2px dashed var(--kazakh-blue);
  border-radius: 20px;
  padding: 4rem 2rem;
  text-align: center;
  color: var(--kazakh-blue);
  box-shadow: var(--shadow-lg);
}

.canvas-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
}

.canvas-placeholder h3 {
  font-size: 1.5rem;
  margin-bottom: 1rem;
  color: var(--kazakh-blue);
}

.canvas-placeholder p {
  margin-bottom: 2rem;
  opacity: 0.8;
}

/* Биография превью */
.bio-preview {
  background: var(--kazakh-gray);
}

.bio-content {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 4rem;
  align-items: start;
}

.bio-summary {
  margin-bottom: 2rem;
}

.bio-paragraph {
  margin-bottom: 1.5rem;
  font-size: 1.1rem;
  line-height: 1.7;
  color: var(--kazakh-dark);
}

.bio-stats {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
}

.stat-card {
  background: var(--kazakh-white);
  padding: 2rem 1.5rem;
  border-radius: 15px;
  text-align: center;
  box-shadow:
    var(--shadow-md),
    0 0 20px rgba(30, 58, 138, 0.1);
  transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  transform-style: preserve-3d;
  border: 1px solid rgba(30, 58, 138, 0.1);
}

.stat-card:hover {
  transform: translateY(-10px) rotateX(5deg) scale(1.02);
  box-shadow:
    var(--shadow-xl),
    0 0 30px rgba(30, 58, 138, 0.2),
    0 0 50px rgba(212, 160, 23, 0.1);
  border-color: rgba(212, 160, 23, 0.3);
}

.stat-number {
  font-size: 2.5rem;
  font-weight: 700;
  color: var(--kazakh-blue);
  margin-bottom: 0.5rem;
}

.stat-label {
  font-size: 0.9rem;
  color: var(--kazakh-dark);
  opacity: 0.8;
}


/* Адаптивность */
@media (max-width: 1024px) {
  .hero-content-wrapper {
    padding: 0 30px;
  }

  .hero-content {
    flex-direction: column;
    gap: 40px;
    text-align: center;
  }

  .hero-title {
    font-size: 3rem;
  }

  .hero-portrait {
    width: 280px;
    height: 360px;
  }

  .bio-content {
    grid-template-columns: 1fr;
    gap: 2rem;
  }

  .bio-stats {
    grid-template-columns: repeat(4, 1fr);
  }
}

@media (max-width: 768px) {
  .hero-content-wrapper {
    padding: 0 20px;
  }

  .hero-content {
    gap: 35px;
  }

  .hero-title {
    font-size: 2.5rem;
    line-height: 1.1;
  }

  .hero-subtitle {
    font-size: 1.25rem;
  }

  .hero-description {
    font-size: 1.1rem;
  }

  .hero-actions {
    justify-content: center;
    gap: 1rem;
  }

  .hero-actions .btn {
    min-width: 140px;
  }

  .hero-portrait {
    width: 250px;
    height: 320px;
  }

  .bio-stats {
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
  }

  .stat-card {
    padding: 1.5rem 1rem;
  }

  .stat-number {
    font-size: 2rem;
  }
}

@media (max-width: 480px) {
  .hero-content-wrapper {
    padding: 0 15px;
  }

  .hero-content {
    gap: 30px;
  }

  .hero-title {
    font-size: 2rem;
    line-height: 1.1;
  }

  .hero-subtitle {
    font-size: 1.1rem;
  }

  .hero-description {
    font-size: 1rem;
  }

  .hero-quote {
    font-size: 1rem;
    padding: 1rem;
    margin-bottom: 1.5rem;
  }

  .hero-actions {
    flex-direction: column;
    gap: 0.75rem;
  }

  .hero-actions .btn {
    width: 100%;
    min-width: auto;
  }

  .hero-portrait {
    width: 200px;
    height: 260px;
  }

  .bio-stats {
    grid-template-columns: 1fr;
    gap: 0.75rem;
  }

  .stat-card {
    padding: 1.25rem 1rem;
  }

  .canvas-container {
    padding: 0 1rem;
  }

  .canvas-placeholder {
    padding: 2rem 1rem;
  }

  .canvas-icon {
    font-size: 3rem;
  }
}
</style>
