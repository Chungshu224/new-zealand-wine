<template>
  <div class="course-navigation">
    <div class="topbar-horizontal">
      <div class="top-left">
        <div class="logo">🍷 紐西蘭葡萄酒學院</div>
        <button class="back-btn" v-if="props.showMapBack" @click="goBackToHome">← 返回首頁</button>
      </div>
      <div class="top-center" v-if="props.showLevelPills">
        <div class="levels-inline">
          <button class="back-btn-inline" @click="goBackToHome">← 返回 Level 選擇</button>
          <button
            v-for="lvl in levels"
            :key="lvl"
            class="level-pill"
            :class="{ active: localSelectedLevel === lvl }"
            :disabled="!props.testMode && unlockedLevelsLocal.length && !unlockedLevelsLocal.includes(lvl)"
            @click="changeLevel(lvl)">
            Level {{ lvl }} {{ localSelectedLevel === lvl ? '✓' : '' }}
          </button>
        </div>
      </div>
      <div class="top-right" v-if="props.showLevelPills">
        <div class="progress-badge">🏆 {{ progressPercentage }}%</div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const props = defineProps({
  modules: {
    type: Array,
    default: () => []
  },
  levels: {
    type: Array,
    default: () => []
  },
  unlockedLevels: {
    type: Array,
    default: () => []
  },
  testMode: {
    type: Boolean,
    default: false
  },
  selectedLevel: {
    type: Number,
    default: 1
  },
  overlay: {
    type: Boolean,
    default: false
  }
  ,
  topMode: {
    type: Boolean,
    default: false
  },
  showLevelPills: {
    type: Boolean,
    default: false
  },
  showMapBack: {
    type: Boolean,
    default: false
  }
})

const emit = defineEmits(['select-lesson', 'change-level', 'back-to-home'])

const isCollapsed = ref(false)
const expandedModules = ref({})
const activeModule = ref(null)
const activeLesson = ref(null)
const completedLessons = ref([])
const localSelectedLevel = ref(props.selectedLevel || 1)

watch(() => props.selectedLevel, (v) => { localSelectedLevel.value = v })

const progressPercentage = computed(() => {
  const totalLessons = props.modules.reduce((sum, module) => sum + module.lessons.length, 0)
  if (totalLessons === 0) return 0
  return Math.round((completedLessons.value.length / totalLessons) * 100)
})

const groupedByLevel = computed(() => {
  const map = {}
  for (const m of props.modules) {
    const lvl = m.level || 0
    if (!map[lvl]) map[lvl] = []
    map[lvl].push(m)
  }
  return map
})

const unlockedLevelsLocal = computed(() => props.unlockedLevels || [])

const goBackToHome = () => {
  emit('back-to-home')
}

const changeLevel = (level) => {
  // prevent selecting locked level (unless testMode active)
  if (!props.testMode) {
    if (level !== 0 && unlockedLevelsLocal.value.length && !unlockedLevelsLocal.value.includes(level)) {
      alert(`Level ${level} 尚未解鎖，請先完成前一級課程。`)
      return
    }
  }
  localSelectedLevel.value = level
  emit('change-level', level)
}

const toggleNav = () => {
  isCollapsed.value = !isCollapsed.value
}

const toggleModule = (moduleId) => {
  expandedModules.value[moduleId] = !expandedModules.value[moduleId]
}

const selectLesson = (moduleId, lesson) => {
  // check module level unlock (unless testMode)
  const module = props.modules.find(m => m.id === moduleId)
  if (!props.testMode) {
    if (module && module.level && unlockedLevelsLocal.value.length && !unlockedLevelsLocal.value.includes(module.level)) {
      alert(`此 Level ${module.level} 尚未解鎖，請先完成前一級課程。`)
      return
    }
  }

  activeModule.value = moduleId
  activeLesson.value = lesson.id
  emit('select-lesson', { moduleId, lesson })
}

const markLessonComplete = (lessonId) => {
  if (!completedLessons.value.includes(lessonId)) {
    completedLessons.value.push(lessonId)
    localStorage.setItem('nz-wine-progress', JSON.stringify(completedLessons.value))
  }
}

const loadProgress = () => {
  const saved = localStorage.getItem('nz-wine-progress')
  if (saved) {
    completedLessons.value = JSON.parse(saved)
  }
}

defineExpose({
  markLessonComplete
})

onMounted(() => {
  loadProgress()
  // Expand first module by default
  if (props.modules.length > 0) {
    expandedModules.value[props.modules[0].id] = true
  }
  // expand first module of each level by default
  props.modules.forEach(m => { if (!expandedModules.value[m.id]) expandedModules.value[m.id] = false })
})
</script>

<style scoped>
.course-navigation {
  width: 100%;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  color: white;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.15);
  position: sticky;
  top: 0;
  z-index: 100;
}

.course-navigation.collapsed {
  width: 60px;
}

.nav-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  background: rgba(0, 0, 0, 0.2);
  border-bottom: 2px solid rgba(255, 255, 255, 0.1);
}

.nav-header h2 {
  margin: 0;
  font-size: 20px;
  font-weight: 600;
}

.level-selector {
  display: flex;
  gap: 8px;
  margin-right: 12px;
}

.level-btn {
  background: rgba(255,255,255,0.08);
  border: 1px solid rgba(255,255,255,0.06);
  color: white;
  padding: 6px 8px;
  border-radius: 6px;
  font-size: 12px;
  cursor: pointer;
}

.level-btn:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}

.level-btn.active {
  background: rgba(255,255,255,0.18);
  border-color: rgba(255,255,255,0.18);
}

.level-lock {
  margin-left: 8px;
  opacity: 0.85;
}

.toggle-btn {
  background: rgba(255, 255, 255, 0.2);
  border: none;
  color: white;
  padding: 8px 12px;
  border-radius: 5px;
  cursor: pointer;
  transition: background 0.2s;
}

.toggle-btn:hover {
  background: rgba(255, 255, 255, 0.3);
}

.nav-content {
  padding: 20px;
}

.course-progress {
  margin-bottom: 25px;
  padding: 15px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 10px;
}

.progress-bar {
  width: 100%;
  height: 8px;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 8px;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #4CAF50, #8BC34A);
  transition: width 0.5s ease;
}

.progress-text {
  margin: 0;
  font-size: 14px;
  text-align: center;
  color: rgba(255, 255, 255, 0.9);
}

.module-item {
  margin-bottom: 15px;
}

.module-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px 15px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
}

.module-header:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateX(3px);
}

.module-header.active {
  background: rgba(255, 255, 255, 0.25);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.module-icon {
  font-size: 24px;
}

.module-title {
  flex: 1;
  font-weight: 600;
  font-size: 15px;
}

.module-arrow {
  font-size: 12px;
  opacity: 0.7;
}

.lessons-list {
  margin-top: 8px;
  margin-left: 20px;
}

.lesson-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 12px;
  margin-bottom: 5px;
  background: rgba(0, 0, 0, 0.2);
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
}

.lesson-item:hover {
  background: rgba(255, 255, 255, 0.15);
  transform: translateX(3px);
}

.lesson-item.active {
  background: rgba(255, 255, 255, 0.2);
  border-left: 3px solid #4CAF50;
}

.lesson-item.completed .lesson-status {
  color: #4CAF50;
  font-weight: bold;
}

.lesson-status {
  font-size: 18px;
  opacity: 0.7;
}

.lesson-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.lesson-title {
  font-size: 14px;
  font-weight: 500;
}

.lesson-duration {
  font-size: 12px;
  opacity: 0.7;
}

/* Scrollbar styling */
.course-navigation::-webkit-scrollbar {
  width: 8px;
}

.course-navigation::-webkit-scrollbar-track {
  background: rgba(0, 0, 0, 0.2);
}

.course-navigation::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 4px;
}

.course-navigation::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.4);
}

.topbar-horizontal {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
  padding: 14px 24px;
  max-width: 1400px;
  margin: 0 auto;
  width: 100%;
  box-sizing: border-box;
}

.top-left {
  display: flex;
  align-items: center;
  gap: 16px;
}

.logo {
  font-weight: 700;
  font-size: 18px;
  white-space: nowrap;
}

.back-btn {
  background: rgba(255,255,255,0.15);
  border: none;
  color: white;
  padding: 6px 12px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 13px;
}

.back-btn-inline {
  background: rgba(255,255,255,0.2);
  border: none;
  color: white;
  padding: 10px 20px;
  border-radius: 24px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.2s;
}

.back-btn-inline:hover {
  background: rgba(255,255,255,0.3);
  transform: translateY(-2px);
}

.top-center {
  flex: 1;
  display: flex;
  justify-content: center;
}

.levels-inline {
  display: flex;
  gap: 12px;
  align-items: center;
}

.level-pill {
  background: rgba(255,255,255,0.15);
  color: white;
  padding: 10px 20px;
  border-radius: 24px;
  border: 2px solid transparent;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: all 0.2s;
}

.level-pill:hover:not(:disabled) {
  background: rgba(255,255,255,0.25);
  transform: translateY(-2px);
}

.level-pill.active {
  background: white;
  color: #667eea;
  font-weight: 700;
  border-color: white;
}

.level-pill:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.top-right {
  display: flex;
  align-items: center;
  gap: 12px;
}

.progress-badge {
  background: rgba(255,255,255,0.2);
  padding: 8px 16px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: 600;
}

@media (max-width: 900px) {
  .topbar-horizontal {
    flex-wrap: wrap;
    padding: 12px 16px;
  }
  
  .top-left .logo {
    font-size: 16px;
  }

  .back-btn, .back-btn-inline {
    padding: 10px 16px;
    font-size: 14px;
    min-height: 44px;
  }
  
  .top-center {
    order: 3;
    width: 100%;
    margin-top: 8px;
  }
  
  .levels-inline {
    width: 100%;
    overflow-x: auto;
    justify-content: flex-start;
    gap: 8px;
    padding-bottom: 4px;
    -webkit-overflow-scrolling: touch;
  }
  
  .level-pill {
    padding: 10px 18px;
    font-size: 14px;
    min-height: 44px;
    white-space: nowrap;
  }

  .progress-badge {
    padding: 8px 14px;
    font-size: 13px;
  }
}

@media (max-width: 480px) {
  .topbar-horizontal {
    padding: 10px 8px;
  }

  .top-left .logo {
    font-size: 14px;
  }

  .back-btn, .back-btn-inline {
    padding: 8px 12px;
    font-size: 13px;
  }

  .level-pill {
    padding: 8px 14px;
    font-size: 13px;
    min-height: 40px;
  }
}
</style>
