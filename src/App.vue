<script setup>
import { ref, provide } from 'vue'
import { RouterView } from 'vue-router'
import Navigation from './components/Navigation.vue'
import Footer from './components/Footer.vue'
import languages from './data/languages.json'

// Текущий язык
const currentLanguage = ref('kk')
const translations = ref(languages)

// Предоставляем переводы и функцию смены языка
provide('translations', translations)
provide('currentLanguage', currentLanguage)

const changeLanguage = (lang) => {
  currentLanguage.value = lang
}
provide('changeLanguage', changeLanguage)
</script>

<template>
  <div id="app">
    <Navigation />
    
    <main class="main-content">
      <RouterView />
    </main>
    
    <Footer />
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&family=Merriweather:wght@300;400;700&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  /* Казахская цветовая палитра */
  --kazakh-blue: #1E3A8A;
  --kazakh-gold: #D4A017;
  --kazakh-green: #15803D;
  --kazakh-white: #FFFFFF;
  --kazakh-gray: #F8FAFC;
  --kazakh-dark: #1F2937;
  
  /* Градиенты */
  --gradient-primary: linear-gradient(135deg, var(--kazakh-blue) 0%, var(--kazakh-gold) 100%);
  --gradient-secondary: linear-gradient(135deg, var(--kazakh-green) 0%, var(--kazakh-blue) 100%);
  
  /* Тени */
  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  --shadow-xl: 0 20px 25px -5px rgba(0, 0, 0, 0.1);
}

body {
  font-family: 'Roboto', sans-serif;
  background-color: var(--kazakh-gray);
  color: var(--kazakh-dark);
  line-height: 1.6;
  min-height: 100vh;
}

#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

.main-content {
  flex: 1;
}

/* Утилитарные классы */
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
}

.section {
  padding: 4rem 0;
}

.section-title {
  font-family: 'Merriweather', serif;
  font-size: 2.5rem;
  font-weight: 700;
  text-align: center;
  margin-bottom: 3rem;
  color: var(--kazakh-blue);
  position: relative;
}

.section-title::after {
  content: '';
  position: absolute;
  bottom: -10px;
  left: 50%;
  transform: translateX(-50%);
  width: 80px;
  height: 4px;
  background: var(--gradient-primary);
  border-radius: 2px;
}

.btn {
  display: inline-block;
  padding: 0.75rem 2rem;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 500;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
  text-align: center;
  position: relative;
  overflow: hidden;
  transform-style: preserve-3d;
}

.btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.3), transparent);
  transition: left 0.5s ease;
}

.btn:hover::before {
  left: 100%;
}

.btn-primary {
  background: var(--gradient-primary);
  color: var(--kazakh-white);
  box-shadow: 
    var(--shadow-md),
    0 0 20px rgba(30, 58, 138, 0.3);
}

.btn-primary:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 
    var(--shadow-lg),
    0 0 30px rgba(30, 58, 138, 0.5),
    0 0 50px rgba(212, 160, 23, 0.2);
}

.btn-secondary {
  background: transparent;
  color: var(--kazakh-blue);
  border: 2px solid var(--kazakh-blue);
  box-shadow: 0 0 10px rgba(30, 58, 138, 0.2);
}

.btn-secondary:hover {
  background: var(--kazakh-blue);
  color: var(--kazakh-white);
  transform: translateY(-3px) scale(1.05);
  box-shadow: 
    var(--shadow-lg),
    0 0 30px rgba(30, 58, 138, 0.5);
}

/* Анимации */
@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes fadeInDown {
  from {
    opacity: 0;
    transform: translateY(-30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideInLeft {
  from {
    opacity: 0;
    transform: translateX(-50px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes slideInRight {
  from {
    opacity: 0;
    transform: translateX(50px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

/* Новые крутые анимации */
@keyframes float {
  0%, 100% {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-10px);
  }
}

@keyframes pulse {
  0%, 100% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
}

@keyframes rotate3D {
  from {
    transform: rotateX(0deg) rotateY(0deg);
  }
  to {
    transform: rotateX(360deg) rotateY(360deg);
  }
}

@keyframes flipInX {
  from {
    opacity: 0;
    transform: perspective(400px) rotateX(90deg);
  }
  to {
    opacity: 1;
    transform: perspective(400px) rotateX(0deg);
  }
}

@keyframes bounceIn {
  0% {
    opacity: 0;
    transform: scale(0.3);
  }
  50% {
    opacity: 1;
    transform: scale(1.05);
  }
  70% {
    transform: scale(0.9);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes shimmer {
  0% {
    background-position: -200% 0;
  }
  100% {
    background-position: 200% 0;
  }
}

@keyframes glow {
  0%, 100% {
    box-shadow: 0 0 5px rgba(212, 160, 23, 0.5);
  }
  50% {
    box-shadow: 0 0 20px rgba(212, 160, 23, 0.8), 0 0 30px rgba(212, 160, 23, 0.6);
  }
}

.animate-fade-in-up {
  animation: fadeInUp 0.8s ease-out;
}

.animate-fade-in-down {
  animation: fadeInDown 0.8s ease-out;
}

.animate-slide-in-left {
  animation: slideInLeft 0.8s ease-out;
}

.animate-slide-in-right {
  animation: slideInRight 0.8s ease-out;
}

/* Новые классы анимаций */
.animate-float {
  animation: float 3s ease-in-out infinite;
}

.animate-pulse {
  animation: pulse 2s ease-in-out infinite;
}

.animate-rotate-3d {
  animation: rotate3D 4s linear infinite;
}

.animate-flip-in-x {
  animation: flipInX 1s ease-out;
}

.animate-bounce-in {
  animation: bounceIn 1s ease-out;
}

.animate-shimmer {
  background: linear-gradient(90deg, transparent, rgba(255,255,255,0.4), transparent);
  background-size: 200% 100%;
  animation: shimmer 2s infinite;
}

.animate-glow {
  animation: glow 2s ease-in-out infinite;
}

/* Адаптивность */
@media (max-width: 1024px) {
  .container {
    padding: 0 1.5rem;
  }
  
  .section {
    padding: 3rem 0;
  }
  
  .section-title {
    font-size: 2rem;
  }
}

@media (max-width: 768px) {
  .container {
    padding: 0 1rem;
  }
  
  .section {
    padding: 2rem 0;
  }
  
  .section-title {
    font-size: 1.75rem;
  }
  
  .btn {
    padding: 0.625rem 1.5rem;
    font-size: 0.9rem;
  }
}

@media (max-width: 480px) {
  .section-title {
    font-size: 1.5rem;
  }
}
</style>
