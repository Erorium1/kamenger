<template>
  <div class="hero-search">
    <div class="container">
      <!-- Анимированная поисковая форма -->
      <div class="search-container" :class="{ 'search-expanded': isSearchExpanded }">
        <div class="search-wrapper">
          <button 
            @click="toggleSearch" 
            class="search-toggle"
            :class="{ 'active': isSearchExpanded }"
          >
            <svg class="search-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor">
              <circle cx="11" cy="11" r="8"></circle>
              <path d="m21 21-4.35-4.35"></path>
            </svg>
            <span class="search-text">Батырларды іздеу</span>
          </button>
          
          <div class="search-form" :class="{ 'show': isSearchExpanded }">
            <div class="search-input-container">
              <input
                v-model="searchQuery"
                @input="performSearch"
                type="text"
                placeholder="Батырдың атын енгізіңіз..."
                class="search-input"
                ref="searchInput"
              />
              <button @click="clearSearch" class="clear-btn" v-if="searchQuery">
                <svg viewBox="0 0 24 24" fill="none" stroke="currentColor">
                  <line x1="18" y1="6" x2="6" y2="18"></line>
                  <line x1="6" y1="6" x2="18" y2="18"></line>
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Результаты поиска с анимацией -->
      <div class="search-results" v-if="isSearchExpanded">
        <div class="results-container" :class="{ 'show': searchQuery }">
          <div v-if="isLoading" class="loading">
            <div class="spinner"></div>
            <p>Іздеу жүріп жатыр...</p>
          </div>
          
          <div v-else-if="filteredHeroes.length > 0" class="heroes-grid">
            <div 
              v-for="(hero, index) in filteredHeroes" 
              :key="hero.id"
              class="hero-card"
              :style="{ animationDelay: `${index * 0.1}s` }"
              @click="selectHero(hero)"
            >
              <div class="hero-avatar">
                <span class="hero-initial">{{ hero.name.charAt(0) }}</span>
              </div>
              <div class="hero-info">
                <h3 class="hero-name">{{ hero.name }}</h3>
                <p class="hero-description">{{ hero.description }}</p>
                <div class="hero-badges">
                  <span class="badge" v-for="badge in hero.badges" :key="badge">
                    {{ badge }}
                  </span>
                </div>
              </div>
            </div>
          </div>
          
          <div v-else-if="searchQuery && !isLoading" class="no-results">
            <div class="no-results-icon">🔍</div>
            <h3>Нәтиже табылмады</h3>
            <p>"{{ searchQuery }}" атымен батыр табылмады</p>
          </div>
        </div>
      </div>

      <!-- Выбранный герой -->
      <div v-if="selectedHero" class="selected-hero" :class="{ 'show': selectedHero }">
        <div class="hero-detail">
          <button @click="closeHeroDetail" class="close-btn">×</button>
          <div class="hero-detail-content">
            <div class="hero-detail-avatar">
              <span class="hero-detail-initial">{{ selectedHero.name.charAt(0) }}</span>
            </div>
            <div class="hero-detail-info">
              <h2>{{ selectedHero.name }}</h2>
              <p class="hero-detail-description">{{ selectedHero.fullDescription }}</p>
              <div class="hero-detail-stats">
                <div class="stat">
                  <span class="stat-label">Жыл:</span>
                  <span class="stat-value">{{ selectedHero.period }}</span>
                </div>
                <div class="stat">
                  <span class="stat-label">Аймақ:</span>
                  <span class="stat-value">{{ selectedHero.region }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, nextTick } from 'vue'

// Реактивные данные
const isSearchExpanded = ref(false)
const searchQuery = ref('')
const isLoading = ref(false)
const selectedHero = ref(null)
const searchInput = ref(null)

// База данных батыров
const heroes = ref([
  {
    id: 1,
    name: 'Абылай хан',
    description: 'Қазақ хандығының көрнекті билеушісі',
    fullDescription: 'Абылай хан - қазақ хандығының тарихындағы ең көрнекті билеушілердің бірі. Ол 18 ғасырда қазақ жерін біріктіріп, орыс империясына қарсы күресті.',
    period: '1711-1781',
    region: 'Орталық Қазақстан',
    badges: ['Хан', 'Дипломат', 'Жеңісші']
  },
  {
    id: 2,
    name: 'Кенесары хан',
    description: 'Қазақ хандығының соңғы хандарының бірі',
    fullDescription: 'Кенесары хан - қазақ хандығының соңғы хандарының бірі. Ол орыс отаршылдығына қарсы күресті және қазақ хандығын қалпына келтіруге тырысты.',
    period: '1802-1847',
    region: 'Оңтүстік Қазақстан',
    badges: ['Хан', 'Күрескер', 'Ұлтшыл']
  },
  {
    id: 3,
    name: 'Амангелді Иманов',
    description: '1916 жылғы ұлт-азаттық көтерілісінің көсемі',
    fullDescription: 'Амангелді Иманов - 1916 жылғы ұлт-азаттық көтерілісінің көсемі. Ол орыс отаршылдығына қарсы қазақ халқының көтерілісін басқарды.',
    period: '1873-1919',
    region: 'Торғай облысы',
    badges: ['Көтерілісші', 'Ұлтшыл', 'Көсем']
  },
  {
    id: 4,
    name: 'Жамбыл Жабаев',
    description: 'Қазақ халқының ақыны',
    fullDescription: 'Жамбыл Жабаев - қазақ халқының атақты ақыны. Ол 19-20 ғасырларда өмір сүріп, қазақ әдебиетіне зор үлес қосты.',
    period: '1846-1945',
    region: 'Жамбыл облысы',
    badges: ['Ақын', 'Әдебиетші', 'Атақты']
  },
  {
    id: 5,
    name: 'Алтынсарин Ыбрай',
    description: 'Қазақ халқының педагогы және ағартушысы',
    fullDescription: 'Алтынсарин Ыбрай - қазақ халқының атақты педагогы және ағартушысы. Ол қазақ тіліндегі бірінші оқулықтарды жазды.',
    period: '1841-1889',
    region: 'Қостанай облысы',
    badges: ['Педагог', 'Ағартушы', 'Жазушы']
  },
  {
    id: 6,
    name: 'Шоқан Уәлиханов',
    description: 'Қазақ халқының ғалымы және саяхатшысы',
    fullDescription: 'Шоқан Уәлиханов - қазақ халқының атақты ғалымы, саяхатшысы және этнографы. Ол Орталық Азияны зерттеді.',
    period: '1835-1865',
    region: 'Солтүстік Қазақстан',
    badges: ['Ғалым', 'Саяхатшы', 'Этнограф']
  }
])

// Фильтрованные герои
const filteredHeroes = computed(() => {
  if (!searchQuery.value.trim()) return []
  
  return heroes.value.filter(hero => 
    hero.name.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
    hero.description.toLowerCase().includes(searchQuery.value.toLowerCase())
  )
})

// Методы
const toggleSearch = async () => {
  isSearchExpanded.value = !isSearchExpanded.value
  
  if (isSearchExpanded.value) {
    await nextTick()
    searchInput.value?.focus()
  } else {
    searchQuery.value = ''
    selectedHero.value = null
  }
}

const performSearch = () => {
  if (searchQuery.value.trim()) {
    isLoading.value = true
    setTimeout(() => {
      isLoading.value = false
    }, 500)
  }
}

const clearSearch = () => {
  searchQuery.value = ''
  selectedHero.value = null
}

const selectHero = (hero) => {
  selectedHero.value = hero
}

const closeHeroDetail = () => {
  selectedHero.value = null
}
</script>

<style scoped>
.hero-search {
  min-height: 100vh;
  padding: 2rem 0;
}

.search-container {
  max-width: 800px;
  margin: 0 auto;
  transition: all 0.3s ease;
}

.search-wrapper {
  position: relative;
}

.search-toggle {
  background: rgba(255, 255, 255, 0.2);
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 50px;
  padding: 1rem 2rem;
  color: white;
  font-size: 1.1rem;
  font-weight: 500;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  transition: all 0.3s ease;
  width: 100%;
  justify-content: center;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.search-toggle:hover {
  background: rgba(255, 255, 255, 0.3);
  transform: translateY(-2px);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.2);
}

.search-toggle.active {
  background: rgba(255, 255, 255, 0.9);
  color: #333;
  border-color: rgba(255, 255, 255, 0.8);
}

.search-icon {
  width: 24px;
  height: 24px;
  stroke-width: 2;
}

.search-form {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(20px);
  border-radius: 20px;
  padding: 1.5rem;
  margin-top: 1rem;
  opacity: 0;
  transform: translateY(-20px);
  transition: all 0.3s ease;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.2);
  z-index: 10;
}

.search-form.show {
  opacity: 1;
  transform: translateY(0);
}

.search-input-container {
  position: relative;
}

.search-input {
  width: 100%;
  padding: 1rem 3rem 1rem 1rem;
  border: 2px solid #e1e5e9;
  border-radius: 15px;
  font-size: 1.1rem;
  outline: none;
  transition: all 0.3s ease;
  background: white;
}

.search-input:focus {
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.clear-btn {
  position: absolute;
  right: 1rem;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  cursor: pointer;
  color: #999;
  padding: 0.5rem;
  border-radius: 50%;
  transition: all 0.2s ease;
}

.clear-btn:hover {
  background: #f0f0f0;
  color: #666;
}

.clear-btn svg {
  width: 20px;
  height: 20px;
  stroke-width: 2;
}

.search-results {
  margin-top: 2rem;
  max-width: 1000px;
  margin-left: auto;
  margin-right: auto;
}

.results-container {
  opacity: 0;
  transform: translateY(20px);
  transition: all 0.3s ease;
}

.results-container.show {
  opacity: 1;
  transform: translateY(0);
}

.loading {
  text-align: center;
  padding: 3rem;
  color: white;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid rgba(255, 255, 255, 0.3);
  border-top: 4px solid white;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 1rem;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.heroes-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
  padding: 1rem;
}

.hero-card {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 1.5rem;
  cursor: pointer;
  transition: all 0.3s ease;
  animation: slideInUp 0.6s ease-out both;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.hero-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.2);
}

@keyframes slideInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.hero-avatar {
  width: 60px;
  height: 60px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 1rem;
}

.hero-initial {
  color: white;
  font-size: 1.5rem;
  font-weight: bold;
}

.hero-name {
  font-size: 1.3rem;
  font-weight: 600;
  color: #333;
  margin-bottom: 0.5rem;
}

.hero-description {
  color: #666;
  line-height: 1.5;
  margin-bottom: 1rem;
}

.hero-badges {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.badge {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  padding: 0.3rem 0.8rem;
  border-radius: 15px;
  font-size: 0.8rem;
  font-weight: 500;
}

.no-results {
  text-align: center;
  padding: 3rem;
  color: white;
}

.no-results-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
}

.no-results h3 {
  font-size: 1.5rem;
  margin-bottom: 0.5rem;
}

.selected-hero {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(5px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  opacity: 0;
  visibility: hidden;
  transition: all 0.3s ease;
}

.selected-hero.show {
  opacity: 1;
  visibility: visible;
}

.hero-detail {
  background: white;
  border-radius: 20px;
  padding: 2rem;
  max-width: 600px;
  width: 90%;
  max-height: 80vh;
  overflow-y: auto;
  position: relative;
  animation: scaleIn 0.3s ease-out;
}

@keyframes scaleIn {
  from {
    transform: scale(0.8);
    opacity: 0;
  }
  to {
    transform: scale(1);
    opacity: 1;
  }
}

.close-btn {
  position: absolute;
  top: 1rem;
  right: 1rem;
  background: none;
  border: none;
  font-size: 2rem;
  cursor: pointer;
  color: #999;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
}

.close-btn:hover {
  background: #f0f0f0;
  color: #666;
}

.hero-detail-content {
  display: flex;
  gap: 1.5rem;
  align-items: flex-start;
}

.hero-detail-avatar {
  width: 80px;
  height: 80px;
  background: linear-gradient(135deg, #667eea, #764ba2);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.hero-detail-initial {
  color: white;
  font-size: 2rem;
  font-weight: bold;
}

.hero-detail-info {
  flex: 1;
}

.hero-detail-info h2 {
  font-size: 1.8rem;
  color: #333;
  margin-bottom: 1rem;
}

.hero-detail-description {
  color: #666;
  line-height: 1.6;
  margin-bottom: 1.5rem;
}

.hero-detail-stats {
  display: flex;
  gap: 2rem;
}

.stat {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}

.stat-label {
  font-size: 0.9rem;
  color: #999;
  font-weight: 500;
}

.stat-value {
  font-size: 1.1rem;
  color: #333;
  font-weight: 600;
}

@media (max-width: 768px) {
  .heroes-grid {
    grid-template-columns: 1fr;
    padding: 0.5rem;
  }
  
  .hero-detail-content {
    flex-direction: column;
    text-align: center;
  }
  
  .hero-detail-stats {
    justify-content: center;
  }
}
</style>
