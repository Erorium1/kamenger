<template>
  <header class="navigation">
    <nav class="nav-container">
      <div class="container">
        <div class="nav-content">
          <!-- Логотип -->
          <div class="logo">
            <router-link to="/" class="logo-link">
              <span class="logo-text">Кемер Онғалбаев</span>
            </router-link>
          </div>

          <!-- Навигационное меню -->
          <ul class="nav-menu" :class="{ 'nav-menu-open': isMenuOpen }">
            <li class="nav-item">
              <router-link to="/" class="nav-link" @click="closeMenu">
                {{ t('nav.home') }}
              </router-link>
            </li>
            <li class="nav-item">
              <router-link to="/biography" class="nav-link" @click="closeMenu">
                {{ t('nav.biography') }}
              </router-link>
            </li>
            <li class="nav-item">
              <router-link to="/gallery" class="nav-link" @click="closeMenu">
                {{ t('nav.gallery') }}
              </router-link>
            </li>
            <!-- <li class="nav-item">
              <router-link to="/legacy" class="nav-link" @click="closeMenu">
                {{ t('nav.legacy') }}
              </router-link>
            </li>
            <li class="nav-item">
              <router-link to="/contact" class="nav-link" @click="closeMenu">
                {{ t('nav.contact') }}
              </router-link>
            </li> -->
          </ul>

          <!-- Переключатель языков -->
          <div class="language-switcher">
            <div class="lang-dropdown">
              <button 
                @click="toggleLangDropdown"
                class="lang-current"
                :title="getCurrentLangName()"
              >
                {{ getCurrentLangFlag() }}
                <svg class="lang-arrow" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                  <path d="m6 9 6 6 6-6"/>
                </svg>
              </button>
              <div class="lang-options" :class="{ 'lang-options-open': isLangDropdownOpen }">
                <button 
                  v-for="lang in languages" 
                  :key="lang.code"
                  @click="handleLanguageChange(lang.code)"
                  class="lang-option"
                  :class="{ 'lang-option-active': currentLanguage === lang.code }"
                  :title="lang.name"
                >
                  {{ lang.flag }} {{ lang.code.toUpperCase() }}
                </button>
              </div>
            </div>
          </div>

          <!-- Мобильное меню кнопка -->
          <button 
            class="mobile-menu-btn"
            @click="toggleMenu"
            :class="{ 'mobile-menu-btn-open': isMenuOpen }"
          >
            <span class="hamburger-line"></span>
            <span class="hamburger-line"></span>
            <span class="hamburger-line"></span>
          </button>
        </div>
      </div>
    </nav>
  </header>
</template>

<script setup>
import { ref, inject, computed } from 'vue'

// Инъекция переводов и текущего языка
const translations = inject('translations')
const currentLanguage = inject('currentLanguage')
const changeLanguage = inject('changeLanguage')

// Состояние мобильного меню
const isMenuOpen = ref(false)

// Состояние выпадающего меню языков
const isLangDropdownOpen = ref(false)

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

// Переключение мобильного меню
const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value
  isLangDropdownOpen.value = false
}

// Закрытие мобильного меню
const closeMenu = () => {
  isMenuOpen.value = false
}

// Переключение выпадающего меню языков
const toggleLangDropdown = () => {
  isLangDropdownOpen.value = !isLangDropdownOpen.value
  isMenuOpen.value = false
}

// Получение флага текущего языка
const getCurrentLangFlag = () => {
  const lang = languages.find(l => l.code === currentLanguage.value)
  return lang ? lang.flag : '🌐'
}

// Получение названия текущего языка
const getCurrentLangName = () => {
  const lang = languages.find(l => l.code === currentLanguage.value)
  return lang ? lang.name : 'Language'
}

// Смена языка
const handleLanguageChange = (langCode) => {
  changeLanguage(langCode)
  isLangDropdownOpen.value = false
  closeMenu()
}

// Закрытие выпадающего меню при клике вне его
const handleClickOutside = (event) => {
  if (!event.target.closest('.lang-dropdown')) {
    isLangDropdownOpen.value = false
  }
}
</script>

<style scoped>
.navigation {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(30, 58, 138, 0.1);
  box-shadow: var(--shadow-sm);
}

.nav-container {
  padding: 1rem 0;
}

.nav-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.logo {
  flex-shrink: 0;
}

.logo-link {
  text-decoration: none;
  color: var(--kazakh-blue);
  font-family: 'Merriweather', serif;
  font-size: 1.5rem;
  font-weight: 700;
  transition: color 0.3s ease;
}

.logo-link:hover {
  color: var(--kazakh-gold);
}

.nav-menu {
  display: flex;
  align-items: center;
  gap: 2rem;
  list-style: none;
  margin: 0;
  padding: 0;
}

.nav-link {
  text-decoration: none;
  color: var(--kazakh-dark);
  font-weight: 500;
  font-size: 1rem;
  padding: 0.5rem 1rem;
  border-radius: 6px;
  transition: all 0.3s ease;
  position: relative;
}

.nav-link:hover {
  color: var(--kazakh-blue);
  background-color: rgba(30, 58, 138, 0.05);
}

.nav-link.router-link-active {
  color: var(--kazakh-blue);
  background-color: rgba(30, 58, 138, 0.1);
}

.nav-link.router-link-active::after {
  content: '';
  position: absolute;
  bottom: -2px;
  left: 50%;
  transform: translateX(-50%);
  width: 20px;
  height: 2px;
  background: var(--kazakh-gold);
  border-radius: 1px;
}

.language-switcher {
  position: relative;
}

.lang-dropdown {
  position: relative;
}

.lang-current {
  background: none;
  border: 2px solid transparent;
  border-radius: 8px;
  padding: 0.5rem 0.75rem;
  cursor: pointer;
  font-size: 1.1rem;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  min-width: 60px;
  height: 40px;
}

.lang-current:hover {
  border-color: var(--kazakh-blue);
  background-color: rgba(30, 58, 138, 0.05);
}

.lang-arrow {
  width: 16px;
  height: 16px;
  stroke-width: 2;
  transition: transform 0.3s ease;
}

.lang-dropdown:hover .lang-arrow {
  transform: rotate(180deg);
}

.lang-options {
  position: absolute;
  top: 100%;
  right: 0;
  background: var(--kazakh-white);
  border: 1px solid rgba(30, 58, 138, 0.1);
  border-radius: 8px;
  box-shadow: var(--shadow-lg);
  padding: 0.5rem 0;
  min-width: 120px;
  opacity: 0;
  visibility: hidden;
  transform: translateY(-10px);
  transition: all 0.3s ease;
  z-index: 1001;
}

.lang-options-open {
  opacity: 1;
  visibility: visible;
  transform: translateY(0);
}

.lang-option {
  width: 100%;
  background: none;
  border: none;
  padding: 0.75rem 1rem;
  cursor: pointer;
  font-size: 0.9rem;
  text-align: left;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  color: var(--kazakh-dark);
}

.lang-option:hover {
  background-color: rgba(30, 58, 138, 0.05);
  color: var(--kazakh-blue);
}

.lang-option-active {
  background-color: var(--kazakh-blue);
  color: var(--kazakh-white);
}

.mobile-menu-btn {
  display: none;
  flex-direction: column;
  gap: 4px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 6px;
  transition: all 0.3s ease;
}

.mobile-menu-btn:hover {
  background-color: rgba(30, 58, 138, 0.05);
}

.hamburger-line {
  width: 24px;
  height: 2px;
  background-color: var(--kazakh-dark);
  transition: all 0.3s ease;
  border-radius: 1px;
}

.mobile-menu-btn-open .hamburger-line:nth-child(1) {
  transform: rotate(45deg) translate(6px, 6px);
}

.mobile-menu-btn-open .hamburger-line:nth-child(2) {
  opacity: 0;
}

.mobile-menu-btn-open .hamburger-line:nth-child(3) {
  transform: rotate(-45deg) translate(6px, -6px);
}

/* Адаптивность */
@media (max-width: 768px) {
  .nav-menu {
    position: fixed;
    top: 100%;
    left: 0;
    right: 0;
    background: var(--kazakh-white);
    flex-direction: column;
    gap: 0;
    padding: 2rem 0;
    box-shadow: var(--shadow-lg);
    transform: translateY(-100%);
    opacity: 0;
    visibility: hidden;
    transition: all 0.3s ease;
  }

  .nav-menu-open {
    transform: translateY(0);
    opacity: 1;
    visibility: visible;
  }

  .nav-item {
    width: 100%;
    text-align: center;
  }

  .nav-link {
    display: block;
    padding: 1rem 2rem;
    border-radius: 0;
    font-size: 1.1rem;
  }

  .nav-link.router-link-active::after {
    display: none;
  }

  .mobile-menu-btn {
    display: flex;
  }

  .language-switcher {
    order: -1;
  }
  
  .lang-current {
    min-width: 50px;
    padding: 0.5rem;
  }
  
  .lang-arrow {
    width: 14px;
    height: 14px;
  }
}

@media (max-width: 480px) {
  .logo-link {
    font-size: 1.25rem;
  }

  .lang-current {
    min-width: 45px;
    height: 35px;
    font-size: 1rem;
    padding: 0.4rem;
  }
  
  .lang-arrow {
    width: 12px;
    height: 12px;
  }
  
  .lang-options {
    min-width: 100px;
  }
}
</style>
