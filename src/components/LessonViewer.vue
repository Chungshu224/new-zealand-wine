<template>
  <div class="lesson-viewer">
    <!-- Level Overview Page -->
    <div v-if="showLevelOverview" class="level-overview">
      <div class="overview-container">
        <div class="back-to-levels">
          <button class="back-to-levels-btn" @click="$emit('back-to-home')">
            ← 返回 Level 選擇
          </button>
        </div>
        <div class="overview-header">
          <h1>Level {{ selectedLevel }} - {{ getLevelTitle(selectedLevel) }}</h1>
          <p class="overview-subtitle">{{ getLevelDescription(selectedLevel) }}</p>
          <button class="continue-btn" @click="continueLearning">▶ 繼續學習</button>
          <div class="overview-progress">
            <span>已完成 {{ levelCompletedCount }}/{{ levelTotalLessons }} 課程</span>
          </div>
        </div>

        <div class="course-content-section">
          <h2>課程內容</h2>
          <div class="lessons-grid">
            <div v-for="module in currentLevelModules" :key="module.id" class="module-section">
              <div v-for="lesson in module.lessons" :key="lesson.id" 
                   class="lesson-card"
                   :class="{ completed: completedLessons.includes(lesson.id) }"
                   @click="selectLessonFromOverview(lesson)">
                <div class="lesson-card-icon">
                  <span v-if="completedLessons.includes(lesson.id)" class="check-icon">✓</span>
                  <span v-else class="circle-icon">○</span>
                </div>
                <div class="lesson-card-content">
                  <h3>{{ lesson.title }}</h3>
                  <span class="lesson-card-duration">{{ lesson.duration }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-else-if="currentLesson" class="lesson-content">
      <!-- Header -->
      <div class="lesson-header">
        <div class="lesson-meta">
          <h1>{{ currentLesson.title }}</h1>
          <div class="lesson-details">
            <span class="duration">⏱️ {{ currentLesson.duration }}</span>
            <span class="type">📚 {{ getLessonTypeText(currentLesson.type) }}</span>
          </div>
        </div>
        <button class="complete-btn" @click="markComplete" :disabled="isCompleted">
          {{ isCompleted ? '✓ 已完成' : '標記完成' }}
        </button>
      </div>

      <!-- Interactive Map Lesson -->
      <div v-if="currentLesson.type === 'interactive-map'" class="map-lesson">
        <div class="map-container">
          <NewZealandMap 
            :focusRegion="currentLesson.region"
            :showRegionInfo="true"
          />
        </div>
      </div>

      <!-- Review Quiz (must check before slides) -->
      <div v-else-if="currentLesson.type === 'review'" class="review-lesson">
        <ReviewQuiz :config="getReviewConfig()" />
      </div>

      <!-- Slide Lesson -->
      <div v-else-if="currentLesson.type !== 'review' && slides.length > 0" class="slide-lesson">
        <SlideViewer 
          :slides="slides" 
          :currentIndex="currentSlideIndex"
        />
      </div>

      <!-- Text Lesson -->
      <div v-else class="text-lesson">
        <div class="lesson-body" v-html="lessonContent"></div>
      </div>

      <!-- Navigation removed - now in SlideViewer -->
    </div>

    <!-- Welcome / Cover Screen: Level cards -->
    <div v-else class="welcome-screen">
      <div class="cover-container">
        <header class="cover-header">
          <div class="cover-title">
            <h1>紐西蘭葡萄酒學院</h1>
            <p class="cover-sub">依照 Level 分級，搭配互動地圖與課程模組</p>
          </div>
        </header>

        <div class="level-cards">
          <div v-for="lvl of levels" :key="lvl" class="level-card">
            <div class="level-top" :class="`level-top-${lvl}`">
              <div class="level-badge">{{ lvl }}</div>
              <div class="level-info">
                <div class="level-name">{{ getLevelTitle(lvl) }}</div>
                <div class="level-subtitle">Level {{ lvl }}</div>
              </div>
            </div>
            <div class="level-body">
              <p class="level-desc">{{ getLevelDescription(lvl) }}</p>
              <div class="level-stats">
                <div class="stat">
                  <div class="stat-num">{{ statsByLevel[lvl] ? statsByLevel[lvl].lessons : 0 }}</div>
                  <div class="stat-label">堂課</div>
                </div>
                <div class="stat">
                  <div class="stat-num">{{ statsByLevel[lvl] ? statsByLevel[lvl].hours : 0 }}</div>
                  <div class="stat-label">小時</div>
                </div>
                <div class="stat">
                  <div class="stat-num">{{ statsByLevel[lvl] ? statsByLevel[lvl].progress : 0 }}%</div>
                  <div class="stat-label">完成度</div>
                </div>
              </div>
            </div>
            <div class="level-cta">
              <button class="enter-btn" @click="enterLevel(lvl)">開始 Level {{ lvl }}</button>
            </div>
          </div>
        </div>

        <div class="bottom-actions">
          <button class="action-btn explore-map-btn" @click="openMap">
            🗺️ 探索地圖
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import NewZealandMap from './NewZealandMap.vue'
import SlideViewer from './SlideViewer.vue'
import ReviewQuiz from './ReviewQuiz.vue'

const props = defineProps({
  currentLesson: {
    type: Object,
    default: null
  },
  modules: {
    type: Array,
    default: () => []
  },
  completedLessons: {
    type: Array,
    default: () => []
  },
  focusRegion: {
    type: String,
    default: ''
  },
  selectedLevel: {
    type: Number,
    default: 1
  },
  levels: {
    type: Array,
    default: () => []
  },
  slideIndex: {
    type: Number,
    default: 0
  }
})

const emit = defineEmits(['mark-complete', 'navigate', 'start-course', 'enter-level', 'open-map', 'show-level-overview', 'select-lesson-direct', 'update-slide-info', 'back-to-home'])

const showLevelOverview = computed(() => {
  return props.selectedLevel && props.selectedLevel !== 0 && !props.currentLesson
})

const currentLevelModules = computed(() => {
  return props.modules.filter(m => m.level === props.selectedLevel)
})

// expose enter-level
const enterLevel = (lvl) => {
  emit('enter-level', lvl)
}

const continueLearning = () => {
  // Find first incomplete lesson or first lesson
  const levelModules = props.modules.filter(m => m.level === props.selectedLevel)
  for (const module of levelModules) {
    for (const lesson of module.lessons) {
      if (!props.completedLessons.includes(lesson.id)) {
        emit('select-lesson-direct', { moduleId: module.id, lesson })
        return
      }
    }
  }
  // If all completed, show first lesson
  if (levelModules.length > 0 && levelModules[0].lessons.length > 0) {
    emit('select-lesson-direct', { moduleId: levelModules[0].id, lesson: levelModules[0].lessons[0] })
  }
}

const selectLessonFromOverview = (lesson) => {
  const module = props.modules.find(m => m.lessons.some(l => l.id === lesson.id))
  if (module) {
    emit('select-lesson-direct', { moduleId: module.id, lesson })
  }
}

const openMap = () => {
  emit('open-map')
}

const lessonContent = ref('')
const slides = ref([])
const currentSlideIndex = computed(() => props.slideIndex)

const currentSlide = computed(() => {
  return slides.value[currentSlideIndex.value] || null
})

const hasNextSlide = computed(() => {
  return currentSlideIndex.value < slides.value.length - 1
})

const hasPrevSlide = computed(() => {
  return currentSlideIndex.value > 0
})

const nextSlide = () => {
  if (hasNextSlide.value) {
    currentSlideIndex.value++
  }
}

const prevSlide = () => {
  if (hasPrevSlide.value) {
    currentSlideIndex.value--
  }
}

const isCompleted = computed(() => {
  return props.currentLesson && props.completedLessons.includes(props.currentLesson.id)
})

const levelTotalLessons = computed(() => {
  if (!props.selectedLevel || props.selectedLevel === 0) return 0
  return props.modules.reduce((sum, m) => m.level === props.selectedLevel ? sum + m.lessons.length : sum, 0)
})

const levelCompletedCount = computed(() => {
  if (!props.selectedLevel || props.selectedLevel === 0) return 0
  // count completed lessons that belong to modules of selected level
  const levelLessonIds = new Set()
  for (const m of props.modules) {
    if (m.level === props.selectedLevel) {
      for (const l of m.lessons) levelLessonIds.add(l.id)
    }
  }
  return props.completedLessons.filter(id => levelLessonIds.has(id)).length
})

const levelProgressPercentage = computed(() => {
  const total = levelTotalLessons.value
  if (total === 0) return 0
  return Math.round((levelCompletedCount.value / total) * 100)
})

const currentLessonIndex = computed(() => {
  if (!props.currentLesson) return -1
  
  let index = 0
  for (const module of props.modules) {
    for (const lesson of module.lessons) {
      if (lesson.id === props.currentLesson.id) {
        return index
      }
      index++
    }
  }
  return -1
})

const totalLessons = computed(() => {
  return props.modules.reduce((sum, module) => sum + module.lessons.length, 0)
})

const hasPrevious = computed(() => currentLessonIndex.value > 0)
const hasNext = computed(() => currentLessonIndex.value < totalLessons.value - 1)

const getLessonTypeText = (type) => {
  const types = {
    'text': '文字課程',
    'interactive-map': '互動地圖',
    'video': '影片課程',
    'quiz': '測驗',
    'review': '複習測驗'
  }
  return types[type] || '課程'
}

const markComplete = () => {
  if (props.currentLesson) {
    emit('mark-complete', props.currentLesson.id)
  }
}

const goToPrevious = () => {
  emit('navigate', 'previous')
}

const goToNext = () => {
  emit('navigate', 'next')
}

const startCourse = () => {
  emit('start-course')
}

const loadLessonContent = async () => {
  if (!props.currentLesson || props.currentLesson.type === 'interactive-map') {
    lessonContent.value = ''
    slides.value = []
    return
  }

  // Reset slide index
  currentSlideIndex.value = 0

  // Try to load slides JSON first
  try {
    const slidesResponse = await fetch(`/modules/lessons/${props.currentLesson.id}-slides.json`)
    if (slidesResponse.ok) {
      const slidesData = await slidesResponse.json()
      slides.value = slidesData.slides || []
      lessonContent.value = ''
      // Notify parent about slide info
      emit('update-slide-info', {
        current: 0,
        total: slides.value.length,
        hasSlides: true
      })
      return
    }
  } catch (error) {
    // Slides not found, continue to HTML
  }

  // Fallback to HTML content
  slides.value = []
  emit('update-slide-info', { current: 0, total: 0, hasSlides: false })
  try {
    const response = await fetch(`/modules/lessons/${props.currentLesson.id}.html`)
    if (response.ok) {
      lessonContent.value = await response.text()
    } else {
      lessonContent.value = `
        <div class="placeholder-content">
          <h2>${props.currentLesson.title}</h2>
          <p>課程內容製作中...</p>
          <p>此課程將涵蓋關於 ${props.currentLesson.title} 的詳細內容。</p>
        </div>
      `
    }
  } catch (error) {
    lessonContent.value = `
      <div class="placeholder-content">
        <h2>${props.currentLesson.title}</h2>
        <p>課程內容製作中...</p>
      </div>
    `
  }
}

watch(() => props.currentLesson, () => {
  loadLessonContent()
}, { immediate: true })

const statsByLevel = computed(() => {
  const map = {}
  for (const lvl of props.levels) {
    const modules = props.modules.filter(m => m.level === lvl)
    const lessons = modules.flatMap(m => m.lessons || [])
    const lessonIds = lessons.map(l => l.id)
    const completed = props.completedLessons.filter(id => lessonIds.includes(id)).length
    const hours = lessons.reduce((s, l) => {
      // try to parse duration if provided as number in l.duration, fallback to 0.5h per lesson
      const num = typeof l.duration === 'number' ? l.duration : (typeof l.duration === 'string' ? parseFloat(l.duration) : NaN)
      return s + (isNaN(num) ? 0.5 : num)
    }, 0)
    map[lvl] = {
      lessons: lessons.length,
      hours: Math.round(hours * 10) / 10,
      progress: lessons.length ? Math.round((completed / lessons.length) * 100) : 0
    }
  }
  return map
})

const getLevelTitle = (lvl) => {
  const titles = {
    1: '基礎入門',
    2: '中級進階',
    3: '高級專業',
    4: '專家認證'
  }
  return titles[lvl] || `Level ${lvl}`
}

const getLevelDescription = (lvl) => {
  const descriptions = {
    1: '建立紐西蘭葡萄酒的基礎認知，了解地理環境、主要品種和基本釀造工藝',
    2: '深入了解紐西蘭各產區特色與風格差異，掌握專業品鑑與分析技能',
    3: '掌握複雜的風土條件與品質評估，深入理解氣候變遷和市場趋勢',
    4: '專業分析與綜合評估能力培養，成為紐西蘭葡萄酒領域的專業顧問'
  }
  return descriptions[lvl] || '以該等級相關模組組成的課程，包含基礎到進階主題。'
}

const getReviewConfig = () => {
  if (!props.currentLesson) return {}
  
  // 根據不同的 review lesson 返回對應配置
  const configs = {
    'lesson-01-review': {
      bankPath: '/modules/lessons/lesson-01-review-questions.json',
      numQuestions: 10,
      passingScore: 80,
      storageKey: 'nz-wine-level1-review',
      passedKey: 'nz-wine-level1-passed',
      nextLevel: 2
    },
    'lesson-03-review': {
      bankPath: '/modules/lessons/lesson-03-review-questions.json',
      numQuestions: 15,
      passingScore: 80,
      storageKey: 'nz-wine-level2-review',
      passedKey: 'nz-wine-level2-passed',
      nextLevel: 3
    },
    'lesson-05-review': {
      bankPath: '/modules/lessons/lesson-05-review-questions.json',
      numQuestions: 20,
      passingScore: 80,
      storageKey: 'nz-wine-level3-review',
      passedKey: 'nz-wine-level3-passed',
      nextLevel: 4
    }
  }
  
  return configs[props.currentLesson.id] || {
    bankPath: `/modules/lessons/${props.currentLesson.id}-questions.json`,
    numQuestions: 10,
    passingScore: 80
  }
}
</script>


<style scoped>
.lesson-viewer {
  flex: 1;
  width: 100%;
  background: #f8f9fa;
  display: flex;
  flex-direction: column;
  color: #2d3748;
  padding-bottom: 60px;
  overflow-y: auto;
  overflow-x: hidden;
}

.lesson-content {
  max-width: 100%;
  width: 100%;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
}

.lesson-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 32px;
  gap: 20px;
  padding: 0 40px 24px 40px;
  border-bottom: 2px solid #e2e8f0;
}

.lesson-meta h1 {
  margin: 0 0 12px 0;
  font-size: 36px;
  color: #2d3748;
  font-weight: 700;
  line-height: 1.3;
}

.lesson-details {
  display: flex;
  gap: 16px;
  font-size: 14px;
  color: #718096;
  flex-wrap: wrap;
}

.level-badge {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 6px 14px;
  border-radius: 16px;
  font-size: 13px;
  font-weight: 600;
}

.complete-btn {
  padding: 14px 28px;
  background: linear-gradient(135deg, #48bb78 0%, #38a169 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  box-shadow: 0 4px 12px rgba(72, 187, 120, 0.3);
}

.complete-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(72, 187, 120, 0.4);
}

.complete-btn:disabled {
  background: #a0aec0;
  cursor: not-allowed;
  box-shadow: none;
}

.map-lesson {
  background: white;
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
}

.map-container {
  height: 600px;
  width: 100%;
}

.slide-lesson {
  flex: 1;
  display: flex;
  flex-direction: column;
  background: white;
  border-radius: 0;
  overflow: hidden;
  box-shadow: none;
}

.text-lesson {
  background: white;
  border-radius: 16px;
  padding: 40px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
}

.lesson-body {
  line-height: 1.7;
  font-size: 15px;
  color: #222;
}

.lesson-body :deep(h2) {
  color: #133b25;
  margin-top: 22px;
  margin-bottom: 10px;
}

.lesson-body :deep(h3) {
  color: #215432;
  margin-top: 18px;
  margin-bottom: 8px;
}

.placeholder-content {
  text-align: center;
  padding: 40px 16px;
  color: #666;
}

.lesson-navigation {
  display: flex;
  justify-content: space-between;
  margin-top: 40px;
  padding-top: 24px;
  border-top: 2px solid #e2e8f0;
  gap: 12px;
}

.nav-btn {
  padding: 12px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.nav-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(102, 126, 234, 0.4);
}

.next-btn { margin-left: auto }

.welcome-screen {
  width: 100%;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  background: #f8f9fa;
  padding: 60px 20px;
  box-sizing: border-box;
}

.welcome-content {
  max-width: 900px;
  width: 100%;
  margin: 0 auto;
  padding: 36px 20px;
}

.welcome-content h1 {
  font-size: 34px;
  color: #133b25;
  text-align: center;
  margin-bottom: 8px;
}

.subtitle { font-size: 16px; color: #666; text-align: center; margin-bottom: 28px }

.features { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 18px; margin-bottom: 36px }

.feature-card { background: white; padding: 18px; border-radius: 10px; text-align: center; box-shadow: 0 2px 10px rgba(0,0,0,0.04) }

.feature-icon { font-size: 36px; margin-bottom: 8px }

.course-overview { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 2px 10px rgba(0,0,0,0.04); margin-bottom: 20px }

.course-overview h2 { color: #133b25; text-align: center; margin-bottom: 18px; font-size: 22px }

.module-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 12px }

.module-card { padding: 18px; background: #fff; border-radius: 8px; border-left: 3px solid #2f8b4e }

.module-card .module-icon { font-size: 28px; margin-bottom: 6px; display: block }

.module-card h4 { color: #133b25; margin-bottom: 6px; font-size: 15px }

.module-card p { color: #666; font-size: 13px; line-height: 1.5; margin-bottom: 8px }

.lesson-count { display: inline-block; padding: 3px 10px; background: #2f8b4e; color: white; border-radius: 10px; font-size: 12px }

.start-btn { display: block; margin: 12px auto 0; padding: 12px 30px; background: #2f8b4e; color: white; border: none; border-radius: 22px; font-size: 15px }

/* Lessons grid and cards layout improvements */
.lessons-grid {
  display: grid;
  grid-template-columns: repeat(1, 1fr);
  gap: 16px;
  align-items: start;
}

.module-section {
  /* let lesson-card children participate directly in the grid */
  display: contents;
}

.lesson-card {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px;
  background: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 14px rgba(15, 23, 42, 0.06);
  cursor: pointer;
  transition: transform 0.12s ease, box-shadow 0.12s ease, border-color 0.12s ease;
  border: 1px solid transparent;
}

.lesson-card:hover { transform: translateY(-4px); box-shadow: 0 8px 26px rgba(15,23,42,0.08) }

.lesson-card-icon { width: 48px; height: 48px; display:flex; align-items:center; justify-content:center; border-radius: 50%; background: #f1f5f9; color: #2d3748; font-weight:700 }
.lesson-card .check-icon { background: linear-gradient(135deg,#48bb78 0%,#38a169 100%); color: #fff; width: 36px; height: 36px; display:flex; align-items:center; justify-content:center; border-radius: 50%; }
.lesson-card .circle-icon { color: #9aa6b2 }

.lesson-card-content h3 { margin: 0; font-size: 16px; color: #133b25 }
.lesson-card-duration { font-size: 13px; color: #718096 }

.lesson-card.completed { border-color: #48bb78; box-shadow: 0 6px 18px rgba(56,161,105,0.08) }

@media (min-width: 600px) {
  .lessons-grid { grid-template-columns: repeat(2, 1fr); }
}

@media (min-width: 992px) {
  .lessons-grid { grid-template-columns: repeat(3, 1fr); }
}

@media (max-width: 768px) {
  .map-container { height: 360px }
  .lesson-content { padding: 14px }
  .welcome-content { padding: 20px }
  .welcome-content h1 { font-size: 26px }
}
</style>

<style scoped>
/* Cover / level cards styles */
.cover-container {
  max-width: 1200px;
  width: 100%;
  margin: 0 auto;
  padding: 0;
}

.cover-header {
  text-align: center;
  margin-bottom: 48px;
}

.cover-title h1 {
  margin: 0 0 12px 0;
  font-size: 42px;
  color: #2d3748;
  font-weight: 700;
}

.cover-sub {
  margin: 0;
  color: #718096;
  font-size: 18px;
  font-weight: 400;
}

.level-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 24px;
  margin-bottom: 40px;
}

.level-card {
  background: white;
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  overflow: hidden;
  display: flex;
  flex-direction: column;
  transition: transform 0.3s, box-shadow 0.3s;
}

.level-card:hover {
  transform: translateY(-8px);
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.12);
}

.level-top {
  padding: 28px 24px 20px;
  display: flex;
  align-items: center;
  gap: 16px;
  border-bottom: 1px solid #e2e8f0;
}

.level-badge {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 24px;
  color: white;
  flex-shrink: 0;
}

.level-top-1 .level-badge { background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); }
.level-top-2 .level-badge { background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%); }
.level-top-3 .level-badge { background: linear-gradient(135deg, #fa709a 0%, #fee140 100%); }
.level-top-4 .level-badge { background: linear-gradient(135deg, #30cfd0 0%, #330867 100%); }

.level-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.level-name {
  font-weight: 700;
  font-size: 20px;
  color: #2d3748;
  line-height: 1.2;
}

.level-subtitle {
  font-size: 14px;
  color: #718096;
  font-weight: 500;
}

.level-body {
  padding: 24px;
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.level-desc {
  color: #4a5568;
  font-size: 15px;
  line-height: 1.7;
  margin: 0;
  min-height: 48px;
}

.level-stats {
  display: flex;
  gap: 12px;
}

.stat {
  flex: 1;
  background: #f7fafc;
  padding: 12px;
  border-radius: 10px;
  text-align: center;
}

.stat-num {
  font-weight: 700;
  font-size: 20px;
  color: #2d3748;
  display: block;
  margin-bottom: 4px;
}

.stat-label {
  font-size: 12px;
  color: #a0aec0;
}

.level-cta {
  padding: 20px 24px;
  border-top: 1px solid #e2e8f0;
}

.enter-btn {
  width: 100%;
  padding: 14px;
  border-radius: 10px;
  border: none;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
}

.enter-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 16px rgba(102, 126, 234, 0.4);
}

@media (max-width: 900px) {
  .cover-title h1 {
    font-size: 32px;
  }
  
  .level-cards {
    grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
    gap: 20px;
  }
}

.level-overview {
  flex: 1;
  width: 100%;
  background: #f8f9fa;
  padding: 40px 20px 60px;
  box-sizing: border-box;
  overflow: visible;
}

.overview-container {
  max-width: 1200px;
  margin: 0 auto;
  padding-bottom: 20px;
}

.back-to-levels {
  margin-bottom: 24px;
}

.back-to-levels-btn {
  padding: 10px 20px;
  background: white;
  color: #667eea;
  border: 2px solid #667eea;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
}

.back-to-levels-btn:hover {
  background: #667eea;
  color: white;
  transform: translateX(-4px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

.overview-header {
  text-align: center;
  margin-bottom: 48px;
}

.overview-header h1 {
  font-size: 36px;
  color: #2d3748;
  margin: 0 0 12px 0;
}

.overview-subtitle {
  font-size: 18px;
  color: #718096;
  margin: 0 0 24px 0;
}

.continue-btn {
  padding: 16px 48px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 28px;
  font-size: 18px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
  margin-bottom: 16px;
}

.continue-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.4);
}

.overview-progress {
  font-size: 16px;
  color: #4a5568;
  font-weight: 600;
}

.course-content-section {
  background: white;
  border-radius: 16px;
  padding: 40px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
}

@media (max-width: 768px) {
  .course-content-section {
    padding: 24px;
  }
}

.course-content-section h2 {
  font-size: 24px;
  color: #2d3748;
  margin: 0 0 24px 0;
  font-weight: 700;
}

.lessons-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 20px;
}

@media (max-width: 900px) {
  .lessons-grid {
    grid-template-columns: 1fr;
  }
}

.lesson-card {
  display: flex;
  align-items: center;
  gap: 20px;
  padding: 24px;
  background: #f7fafc;
  border: 2px solid #e2e8f0;
  border-radius: 14px;
  cursor: pointer;
  transition: all 0.3s;
  min-height: 96px;
}

.lesson-card:hover {
  border-color: #667eea;
  background: #edf2f7;
  transform: translateY(-3px);
  box-shadow: 0 6px 20px rgba(102, 126, 234, 0.25);
}

.lesson-card.completed {
  background: #e6ffed;
  border-color: #48bb78;
}

.lesson-card.completed:hover {
  border-color: #38a169;
  background: #d4f4dd;
  box-shadow: 0 6px 20px rgba(72, 187, 120, 0.25);
}

.lesson-card-icon {
  width: 56px;
  height: 56px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  font-size: 28px;
  font-weight: bold;
}

.lesson-card:not(.completed) .lesson-card-icon {
  background: white;
  color: #a0aec0;
  border: 2px solid #e2e8f0;
}

.lesson-card.completed .lesson-card-icon {
  background: #48bb78;
  color: white;
}

.lesson-card-content {
  flex: 1;
}

.lesson-card-content h3 {
  margin: 0 0 6px 0;
  font-size: 17px;
  color: #2d3748;
  font-weight: 600;
  line-height: 1.4;
}

.lesson-card-duration {
  font-size: 15px;
  color: #718096;
  font-weight: 500;
}

.bottom-actions {
  margin-top: 48px;
  padding: 32px;
  background: white;
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  display: flex;
  justify-content: center;
  gap: 16px;
}

.action-btn {
  padding: 16px 48px;
  border-radius: 28px;
  border: none;
  font-size: 16px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.action-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.15);
}

.explore-map-btn {
  background: linear-gradient(135deg, #06b6d4 0%, #0891b2 100%);
  color: white;
}

@media (max-width: 600px) {
  .cover-title h1 {
    font-size: 28px;
  }
  
  .cover-sub {
    font-size: 16px;
  }
  
  .level-cards {
    grid-template-columns: 1fr;
  }
  
  .bottom-actions {
    margin-top: 32px;
    padding: 20px;
  }
  
  .action-btn {
    padding: 14px 32px;
    font-size: 15px;
  }
}
</style>

