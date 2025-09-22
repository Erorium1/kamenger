<template>
  <div class="biography">
    <!-- Өмірбаяны: Профиль -->
    <section class="section">
      <div class="container">
        <h1 class="section-title animate-fade-in-down">Өмірбаяны</h1>

        <div class="profile-card animate-bounce-in">
          <div class="profile-content">
            <div class="profile-bullets">
              <p class="bio-paragraph">{{ t('biography.profile.p1') }}</p>
              <p class="bio-paragraph">{{ t('biography.profile.p2') }}</p>
              <p class="bio-paragraph"><strong>{{ t('biography.profile.p3') }}</strong></p>
              <p class="bio-paragraph">{{ t('biography.profile.p4') }}</p>
              <p class="bio-paragraph">{{ t('biography.profile.p5') }}</p>
            </div>
          </div>
        </div>

        <div class="bio-section animate-slide-in-left">
          <h2 class="bio-heading">Жауынгерлік жолы</h2>
          <p class="bio-paragraph">
            Кемер Оңғалбаев армияға 1943 жылы тамыз айында Красноводск қалалық әскери комиссариатынан шақырылды. 18 жастағы жігіт қысқа мерзімді әскери даярлықтан өтісімен 1943 жылдың қазан айында 4-Украина майданының 581-атқыштар полкінің құрамында соғысқа кірісті.
          </p>
        </div>

        <!-- “Шел солдат” бөлімі -->
        <div class="bio-section animate-slide-in-right">
          <h2 class="bio-heading">“Шел солдат”</h2>
          <p class="bio-paragraph">
            Жеңістің 30 жылдығы қарсаңында Одақтық кинематография комитеті «Шел солдат» деген атпен деректі фильм шығаруға кіріседі. Осы фильмді түсіру барысында Деректі фильмдер студиясы «Даңқ» орденінің толық иегері Кемер Оңғалбаевқа сауалнама жолдап, қанағаттанарлықтай жауаптар алады. Бұл жөнінде әйгілі майдангер-жазушы Константин Симонов Кемерге арнайы хат та жолдаған:
          </p>
          <blockquote class="bio-quote">
            <p>«Қадірлі майдандас досым, Кемер!</p>
            <p>Сені алда келе жатқан Ұлы Жеңістің 30 жылдық мерекесімен кұттықтаймын, мықты денсаулық, ұзақ ғұмыр тілеймін. Жеңіс мерекесіне арнап «Шел солдат» атты деректі телефильм түсіргенімізден өзің хабарлысың.Бұл фильм мереке қарсаңында теледидар арқылы және кинозалдарда көрсетіледі. Онда Сен жайында да деректі сюжеттер бар. Қарарсың. Біздердің, сол сұрапыл жылдар жауынгерлерінің, Ұлы Отанымыздың азаттығы жолындағы ерлік қимылдарымыз ешқашан ұмытылмайды.</p>
            <p>Кемер! Өзіңе саулық, семьяңа бақыт тілеймін.</p>
            <footer>Сәлеммен, Константин СИМОНОВ, КСРО Кинематографистер Одағынын төрағасы. Москва. 15.04.1975 ж.»</footer>
          </blockquote>
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
import { inject } from 'vue'
const translations = inject('translations')
const currentLanguage = inject('currentLanguage')
const t = (key) => {
  const keys = key.split('.')
  let result = translations.value[currentLanguage.value]
  for (const k of keys) {
    result = result?.[k]
  }
  return result || key
}
</script>

<style scoped>
.profile-card {
  background: var(--kazakh-white);
  border: 1px solid rgba(30, 58, 138, 0.12);
  border-radius: 14px;
  box-shadow: var(--shadow-lg);
  padding: 2rem;
  margin-bottom: 2rem;
}

.profile-content {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

.bio-section {
  background: var(--kazakh-white);
  border: 1px solid rgba(30, 58, 138, 0.08);
  border-radius: 12px;
  box-shadow: var(--shadow-md);
  padding: 2rem;
  margin-bottom: 2rem;
}

.bio-heading {
  font-family: 'Merriweather', serif;
  color: var(--kazakh-blue);
  font-size: 1.5rem;
  margin-bottom: 1rem;
}

.bio-paragraph {
  font-size: 1.05rem;
  color: var(--kazakh-dark);
}

.bio-quote {
  margin-top: 1rem;
  padding: 1.25rem 1.5rem;
  border-left: 4px solid var(--kazakh-gold);
  background: rgba(212, 160, 23, 0.06);
  border-radius: 8px;
}

.bio-quote p {
  margin-bottom: 0.75rem;
}

.bio-quote footer {
  margin-top: 0.5rem;
  font-style: italic;
  color: #555;
}
</style>
