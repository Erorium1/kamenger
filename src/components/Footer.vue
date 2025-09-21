<template>
  <footer class="footer">
    <div class="footer-wrapper">
      <div class="footer-content">
        <!-- Информация о проекте -->
        <div class="footer-section">
          <h3 class="footer-title">{{ t('footer.title') }}</h3>
          <p class="footer-description">
            {{ t('footer.description') }}
          </p>
          <div class="footer-badges">
            <span class="badge">{{ t('footer.hero') }}</span>
            <span class="badge">{{ t('footer.patriot') }}</span>
          </div>
        </div>

        <!-- Навигация -->
        <div class="footer-section">
          <h4 class="footer-subtitle">{{ t('footer.navigation') }}</h4>
          <ul class="footer-nav">
            <li><router-link to="/" class="footer-link">{{ t('nav.home') }}</router-link></li>
            <li><router-link to="/biography" class="footer-link">{{ t('nav.biography') }}</router-link></li>
            <li><router-link to="/gallery" class="footer-link">{{ t('nav.gallery') }}</router-link></li>
            <li><router-link to="/legacy" class="footer-link">{{ t('nav.legacy') }}</router-link></li>
            <li><router-link to="/contact" class="footer-link">{{ t('nav.contact') }}</router-link></li>
          </ul>
        </div>

        <!-- Источники -->
        <div class="footer-section">
          <h4 class="footer-subtitle">{{ t('footer.sources') }}</h4>
          <ul class="footer-sources">
            <li><span class="source-item">"Маңғыстау батырлары"</span></li>
            <li><span class="source-item">"Шел солдат" - К. Симонов</span></li>
            <li><span class="source-item">{{ t('footer.archives') }}</span></li>
          </ul>
        </div>

        <!-- Контакты -->
        <div class="footer-section">
          <h4 class="footer-subtitle">{{ t('footer.contact') }}</h4>
          <div class="footer-contact">
            <p class="contact-item">
              <span class="contact-icon">📧</span>
              {{ t('footer.email') }}
            </p>
            <p class="contact-item">
              <span class="contact-icon">📍</span>
              {{ t('footer.location') }}
            </p>
          </div>
        </div>
      </div>

      <!-- Кнопка "Наверх" -->
      <button 
        @click="scrollToTop" 
        class="scroll-to-top"
        :class="{ 'scroll-to-top-visible': isScrollToTopVisible }"
        :title="t('footer.scrollToTop')"
      >
        <svg class="scroll-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor">
          <path d="m18 15-6-6-6 6"/>
        </svg>
      </button>

      <!-- Копирайт -->
      <div class="footer-bottom">
        <p class="copyright">
          © {{ currentYear }} {{ t('footer.copyright') }}
        </p>
        <div class="footer-languages">
          <span 
            v-for="lang in languages" 
            :key="lang.code"
            @click="changeLanguage(lang.code)"
            class="footer-lang"
            :class="{ 'footer-lang-active': currentLanguage === lang.code }"
          >
            {{ lang.flag }}
          </span>
        </div>
      </div>
    </div>
  </footer>
</template>

<script setup>
import { ref, inject, computed, onMounted, onUnmounted } from 'vue'

// Инъекция переводов и текущего языка
const translations = inject('translations')
const currentLanguage = inject('currentLanguage')
const changeLanguage = inject('changeLanguage')

// Состояние кнопки "Наверх"
const isScrollToTopVisible = ref(false)

// Текущий год
const currentYear = computed(() => new Date().getFullYear())

// Доступные языки
const languages = [
  { code: 'kk', name: 'Қазақша', flag: '🇰🇿' },
  { code: 'ru', name: 'Русский', flag: '🇷🇺' },
  { code: 'en', name: 'English', flag: '🇺🇸' }
]

// Функция перевода
const t = (key) => {
  const keys = key.split('.')
  let result = translations.value[currentLanguage.value]
  
  for (const k of keys) {
    result = result?.[k]
  }
  
  return result || key
}

// Обработчик скролла
const handleScroll = () => {
  isScrollToTopVisible.value = window.scrollY > 300
}

// Прокрутка наверх
const scrollToTop = () => {
  window.scrollTo({
    top: 0,
    behavior: 'smooth'
  })
}

// Жизненный цикл
onMounted(() => {
  window.addEventListener('scroll', handleScroll)
})

onUnmounted(() => {
  window.removeEventListener('scroll', handleScroll)
})
</script>

<style scoped>
.footer {
  background: var(--gradient-secondary);
  color: var(--kazakh-white);
  padding: 3rem 0 1rem;
  position: relative;
  margin-top: auto;
  width: 100vw;
  margin-left: calc(-50vw + 50%);
}

.footer-wrapper {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.footer-content {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 2rem;
  margin-bottom: 2rem;
}

.footer-section {
  display: flex;
  flex-direction: column;
}

.footer-title {
  font-family: 'Merriweather', serif;
  font-size: 1.5rem;
  font-weight: 700;
  margin-bottom: 1rem;
  color: var(--kazakh-white);
}

.footer-subtitle {
  font-size: 1.1rem;
  font-weight: 600;
  margin-bottom: 1rem;
  color: var(--kazakh-white);
}

.footer-description {
  line-height: 1.6;
  margin-bottom: 1rem;
  opacity: 0.9;
}

.footer-badges {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.badge {
  background: rgba(255, 255, 255, 0.2);
  padding: 0.25rem 0.75rem;
  border-radius: 15px;
  font-size: 0.85rem;
  font-weight: 500;
  backdrop-filter: blur(10px);
}

.footer-nav,
.footer-sources {
  list-style: none;
  padding: 0;
  margin: 0;
}

.footer-nav li,
.footer-sources li {
  margin-bottom: 0.5rem;
}

.footer-link {
  color: var(--kazakh-white);
  text-decoration: none;
  opacity: 0.9;
  transition: all 0.3s ease;
  padding: 0.25rem 0;
  display: block;
}

.footer-link:hover {
  opacity: 1;
  color: var(--kazakh-gold);
  transform: translateX(5px);
}

.source-item {
  opacity: 0.9;
  font-style: italic;
}

.footer-contact {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.contact-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  opacity: 0.9;
}

.contact-icon {
  font-size: 1.1rem;
}

.scroll-to-top {
  position: fixed;
  bottom: 2rem;
  right: 2rem;
  width: 50px;
  height: 50px;
  background: var(--kazakh-gold);
  border: none;
  border-radius: 50%;
  color: var(--kazakh-white);
  cursor: pointer;
  opacity: 0;
  visibility: hidden;
  transform: translateY(20px);
  transition: all 0.3s ease;
  box-shadow: var(--shadow-lg);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.scroll-to-top:hover {
  background: var(--kazakh-blue);
  transform: translateY(-2px);
  box-shadow: var(--shadow-xl);
}

.scroll-to-top-visible {
  opacity: 1;
  visibility: visible;
  transform: translateY(0);
}

.scroll-icon {
  width: 24px;
  height: 24px;
  stroke-width: 2;
}

.footer-bottom {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 2rem;
  border-top: 1px solid rgba(255, 255, 255, 0.2);
}

.copyright {
  opacity: 0.8;
  font-size: 0.9rem;
}

.footer-languages {
  display: flex;
  gap: 0.5rem;
}

.footer-lang {
  cursor: pointer;
  font-size: 1.2rem;
  padding: 0.25rem;
  border-radius: 4px;
  transition: all 0.3s ease;
  opacity: 0.7;
}

.footer-lang:hover {
  opacity: 1;
  background: rgba(255, 255, 255, 0.1);
}

.footer-lang-active {
  opacity: 1;
  background: rgba(255, 255, 255, 0.2);
}

/* Адаптивность */
@media (max-width: 768px) {
  .footer {
    padding: 2rem 0 1rem;
  }

  .footer-wrapper {
    padding: 0 15px;
  }

  .footer-content {
    grid-template-columns: 1fr;
    gap: 1.5rem;
  }

  .footer-bottom {
    flex-direction: column;
    gap: 1rem;
    text-align: center;
  }

  .scroll-to-top {
    bottom: 1rem;
    right: 1rem;
    width: 45px;
    height: 45px;
  }

  .scroll-icon {
    width: 20px;
    height: 20px;
  }
}

@media (max-width: 480px) {
  .footer-title {
    font-size: 1.25rem;
  }

  .footer-subtitle {
    font-size: 1rem;
  }

  .footer-badges {
    gap: 0.25rem;
  }

  .badge {
    font-size: 0.75rem;
    padding: 0.2rem 0.5rem;
  }
}
</style>
