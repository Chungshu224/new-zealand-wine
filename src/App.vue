<template>
  <div class="app">
    <!-- Test Mode Banner -->
    <div v-if="testMode" class="test-mode-banner">
      <span>🧪 測試模式已啟用 - 所有功能已解鎖</span>
      <button class="close-test-btn" @click="toggleTestMode">退出測試模式</button>
    </div>

    <!-- Top Navigation Bar -->
    <CourseNavigation 
      v-if="!mapMode || currentLesson"
      ref="navRef"
      :modules="filteredModules"
      :levels="levels"
      :unlockedLevels="unlockedLevels"
      :selectedLevel="selectedLevel"
      :testMode="testMode"
      :overlay="false"
      :topMode="true"
      :showLevelPills="false"
      :showMapBack="mapMode && !currentLesson"
      @select-lesson="handleSelectLesson"
      @change-level="val => selectedLevel.value = val"
      @back-to-home="handleBackToHome"
    />

    <div class="main-area">
      <div v-if="currentLesson" class="utility-bar">
        <button class="util-btn" @click="selectedLevel === 1 ? handleBackToLevel1() : (selectedLevel === 2 ? handleBackToLevel2() : (selectedLevel === 3 ? handleBackToLevel3() : handleBackToHome()))">{{ selectedLevel === 1 ? '← 返回Level1課程' : (selectedLevel === 2 ? '← 返回Level2課程' : (selectedLevel === 3 ? '← 返回Level3課程' : '← 返回課程選擇')) }}</button>
        <div class="slide-nav-controls" v-if="showSlideControls">
          <button class="slide-nav-btn" @click="handlePrevSlide" :disabled="!canPrevSlide">
            ← 上一頁
          </button>
          <span class="slide-progress">{{ currentSlideNum }} / {{ totalSlides }}</span>
          <button class="slide-nav-btn" @click="handleNextSlide" :disabled="!canNextSlide">
            下一頁 →
          </button>
        </div>
        <button v-if="!testMode" class="util-btn" @click="toggleTestMode" style="background: #555">測試模式</button>
      </div>

      <NewZealandMap v-if="mapMode && !currentLesson" @back-to-course="handleBackToHome" />
      <LessonViewer v-else
        :currentLesson="currentLesson"
        :modules="courseModules"
        :completedLessons="completedLessons"
        :selectedLevel="selectedLevel"
        :levels="levels"
        :testMode="testMode"
        :focusRegion="focusRegion"
        :slideIndex="slideInfo.current"
        @mark-complete="handleMarkComplete"
        @navigate="handleNavigate"
        @start-course="handleStartCourse"
        @enter-level="handleEnterLevel"
        @open-map="openMapMode"
        @select-lesson-direct="handleSelectLessonDirect"
        @update-slide-info="updateSlideInfo"
        @back-to-home="handleBackToHome"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import CourseNavigation from './components/CourseNavigation.vue'
import LessonViewer from './components/LessonViewer.vue'
import NewZealandMap from './components/NewZealandMap.vue'

const navRef = ref(null)
let selectedLevel = ref(0)
const testMode = ref(false)
const courseModules = ref([])
const currentLesson = ref(null)
const completedLessons = ref([])
const mapMode = ref(false)
const focusRegion = ref(null)

// Slide control states
const slideInfo = ref({ current: 0, total: 0, hasSlides: false })
const showSlideControls = computed(() => slideInfo.value.hasSlides)
const currentSlideNum = computed(() => slideInfo.value.current + 1)
const totalSlides = computed(() => slideInfo.value.total)
const canPrevSlide = computed(() => slideInfo.value.current > 0)
const canNextSlide = computed(() => slideInfo.value.current < slideInfo.value.total - 1)

const levels = computed(() => {
  const set = new Set(courseModules.value.map(m => m.level).filter(l => typeof l !== 'undefined'))
  return Array.from(set).sort((a,b) => a - b)
})

const filteredModules = computed(() => {
  if (!selectedLevel.value || selectedLevel.value === 0) return courseModules.value
  return courseModules.value.filter(m => m.level === selectedLevel.value)
})

const unlockedLevels = computed(() => {
  // If test mode is enabled, unlock all levels
  if (testMode.value) return levels.value.slice().sort((a,b) => a-b)
  const levelsArr = levels.value
  const unlocked = new Set()
  if (levelsArr.length === 0) return []
  unlocked.add(levelsArr[0])

  // helper: get lesson ids for levels < n
  const lessonsByLevel = {}
  for (const m of courseModules.value) {
    lessonsByLevel[m.level] = lessonsByLevel[m.level] || []
    lessonsByLevel[m.level].push(...m.lessons.map(l => l.id))
  }

  for (let i = 1; i < levelsArr.length; i++) {
    const lvl = levelsArr[i]
    // check previous levels completed
    let prevLevels = levelsArr.filter(x => x < lvl)
    let allPrevIds = []
    prevLevels.forEach(pl => { if (lessonsByLevel[pl]) allPrevIds.push(...lessonsByLevel[pl]) })
    const allCompleted = allPrevIds.every(id => completedLessons.value.includes(id))
    if (allCompleted) unlocked.add(lvl)
  }
  return Array.from(unlocked).sort((a,b) => a-b)
})

const loadCourseStructure = async () => {
  try {
    const response = await fetch('/modules/course-structure.json')
    if (response.ok) {
      courseModules.value = await response.json()
    }
  } catch (error) {
    console.error('載入課程結構失敗:', error)
  }
}

const loadProgress = () => {
  const saved = localStorage.getItem('nz-wine-progress')
  if (saved) {
    completedLessons.value = JSON.parse(saved)
  }
}

const handleSelectLesson = ({ moduleId, lesson }) => {
  // prevent selecting locked-level lessons
  const module = courseModules.value.find(m => m.id === moduleId)
  if (!testMode.value) {
    if (module && module.level && !unlockedLevels.value.includes(module.level)) {
      alert(`此 Level ${module.level} 尚未解鎖，請先完成前一級課程。`)
      return
    }
  }

  currentLesson.value = lesson
  // If interactive-map lesson, switch to mapMode and focus region
  if (lesson && lesson.type === 'interactive-map') {
    mapMode.value = true
    focusRegion.value = lesson.region
  }
}

const handleMarkComplete = (lessonId) => {
  if (!completedLessons.value.includes(lessonId)) {
    completedLessons.value.push(lessonId)
    localStorage.setItem('nz-wine-progress', JSON.stringify(completedLessons.value))
    
    // Also update navigation component
    if (navRef.value) {
      navRef.value.markLessonComplete(lessonId)
    }
  }
}

const handleNavigate = (direction) => {
  if (!currentLesson.value) return
  
  let allLessons = []
  for (const module of courseModules.value) {
    allLessons.push(...module.lessons)
  }
  
  const currentIndex = allLessons.findIndex(l => l.id === currentLesson.value.id)
  
  if (direction === 'next' && currentIndex < allLessons.length - 1) {
    currentLesson.value = allLessons[currentIndex + 1]
  } else if (direction === 'previous' && currentIndex > 0) {
    currentLesson.value = allLessons[currentIndex - 1]
  }
}

const handleStartCourse = () => {
  if (courseModules.value.length > 0 && courseModules.value[0].lessons.length > 0) {
    currentLesson.value = courseModules.value[0].lessons[0]
  }
}

const toggleMapMode = () => {
  mapMode.value = !mapMode.value
}

const openMapMode = () => {
  mapMode.value = true
  currentLesson.value = null
}

const toggleTestMode = () => {
  testMode.value = !testMode.value
  try {
    localStorage.setItem('nz-wine-test-mode', testMode.value ? '1' : '0')
  } catch (e) {
    console.warn('無法儲存測試模式設定', e)
  }
}

onMounted(async () => {
  await loadCourseStructure()
  loadProgress()
  // load testMode from localStorage
  const tm = localStorage.getItem('nz-wine-test-mode')
  if (tm === '1' || tm === 'true') testMode.value = true
})

const handleEnterLevel = (level) => {
  selectedLevel.value = level
  // Don't set currentLesson - let LessonViewer show level overview first
  currentLesson.value = null
  mapMode.value = false
}

const handleSelectLessonDirect = (data) => {
  const { lesson } = data
  currentLesson.value = lesson
}

const handleBackToHome = () => {
  currentLesson.value = null
  mapMode.value = false
  selectedLevel.value = 0
  slideInfo.value = { current: 0, total: 0, hasSlides: false }
}

const handleBackToLevel1 = () => {
  // Navigate back to Level 1 overview
  currentLesson.value = null
  mapMode.value = false
  selectedLevel.value = 1
  slideInfo.value = { current: 0, total: 0, hasSlides: false }
}

const handleBackToLevel2 = () => {
  // Navigate back to Level 2 overview
  currentLesson.value = null
  mapMode.value = false
  selectedLevel.value = 2
  slideInfo.value = { current: 0, total: 0, hasSlides: false }
}

const handleBackToLevel3 = () => {
  // Navigate back to Level 3 overview
  currentLesson.value = null
  mapMode.value = false
  selectedLevel.value = 3
  slideInfo.value = { current: 0, total: 0, hasSlides: false }
}

const handlePrevSlide = () => {
  if (canPrevSlide.value) {
    slideInfo.value.current--
    // 滾動到頁面頂部
    window.scrollTo({ top: 0, behavior: 'smooth' })
  }
}

const handleNextSlide = () => {
  if (canNextSlide.value) {
    slideInfo.value.current++
    // 滾動到頁面頂部
    window.scrollTo({ top: 0, behavior: 'smooth' })
  }
}

const updateSlideInfo = (info) => {
  slideInfo.value = info
}
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
  font-family: 'Arial', 'Microsoft YaHei', sans-serif;
  background: #f8f9fa;
  height: 100%;
  overflow-x: hidden;
}

.app {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

.test-mode-banner {
  background: linear-gradient(90deg, #ff6b6b, #ff8787);
  color: white;
  padding: 12px 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 16px;
  font-weight: 600;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.close-test-btn {
  background: rgba(255,255,255,0.25);
  border: 1px solid rgba(255,255,255,0.4);
  color: white;
  padding: 6px 14px;
  border-radius: 16px;
  cursor: pointer;
  font-size: 13px;
}

.main-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  width: 100%;
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 20px;
  box-sizing: border-box;
  overflow: visible;
  min-height: 0;
}

.utility-bar {
  display: flex;
  gap: 10px;
  padding: 16px 0;
  align-items: center;
  justify-content: space-between;
}

.util-btn {
  padding: 8px 16px;
  border-radius: 8px;
  background: #5a67d8;
  color: white;
  border: none;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.2s;
}

.util-btn:hover {
  background: #4c51bf;
  transform: translateY(-1px);
}

.slide-nav-controls {
  display: flex;
  align-items: center;
  gap: 16px;
  margin: 0 auto;
}

.slide-nav-btn {
  padding: 10px 24px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 10px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s;
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.3);
}

.slide-nav-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.slide-nav-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  box-shadow: none;
}

.slide-progress {
  font-size: 16px;
  font-weight: 600;
  color: #4a5568;
  padding: 8px 20px;
  background: #f7fafc;
  border-radius: 16px;
  min-width: 80px;
  text-align: center;
}
</style>
