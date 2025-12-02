<template>
  <div class="slide-viewer">
    <!-- Slide Display -->
    <div class="slide-container" v-if="currentSlide">
      <!-- Cover Slide -->
      <div v-if="currentSlide.type === 'cover'" class="slide slide-cover" :style="{ background: currentSlide.background }">
        <div class="slide-cover-content">
          <div class="slide-icon">{{ currentSlide.icon }}</div>
          <h1 class="slide-title">{{ currentSlide.title }}</h1>
          <p class="slide-subtitle">{{ currentSlide.subtitle }}</p>
        </div>
      </div>

      <!-- Content Slide -->
      <div v-else-if="currentSlide.type === 'content'" class="slide slide-content">
        <div class="slide-header">
          <span class="slide-icon">{{ currentSlide.icon }}</span>
          <h2>{{ currentSlide.title }}</h2>
        </div>
        <div class="slide-body">
          <div v-for="(item, index) in currentSlide.content" :key="index">
            <p v-if="item.type === 'text'" class="text-content">{{ item.text }}</p>
            
            <!-- Stats Grid -->
            <div v-if="item.type === 'stats'" class="stats-grid">
              <div v-for="(stat, idx) in item.items" :key="idx" class="stat-card">
                <div class="stat-icon">{{ stat.icon }}</div>
                <div class="stat-label">{{ stat.label }}</div>
                <div class="stat-value">{{ stat.value }}</div>
              </div>
            </div>
          </div>
        </div>
          <div v-if="currentSlide.quiz" class="quiz-wrapper">
            <QuizViewer :quiz="currentSlide.quiz" />
          </div>
          <div v-if="currentSlide.reviewQuiz" class="review-wrapper">
            <ReviewQuiz :config="currentSlide.reviewQuiz" />
          </div>
      </div>

      <!-- Timeline Slide -->
      <div v-else-if="currentSlide.type === 'timeline'" class="slide slide-timeline">
        <div class="slide-header">
          <span class="slide-icon">{{ currentSlide.icon }}</span>
          <h2>{{ currentSlide.title }}</h2>
          <p v-if="currentSlide.subtitle" class="timeline-subtitle">{{ currentSlide.subtitle }}</p>
        </div>
        <div class="timeline">
          <div v-for="(item, index) in currentSlide.items" :key="index" class="timeline-item">
            <div class="timeline-marker"></div>
            <div class="timeline-content">
              <div class="timeline-year">{{ item.year }}</div>
              <div class="timeline-text">{{ item.text }}</div>
            </div>
          </div>
        </div>
        <div v-if="currentSlide.quiz" class="quiz-wrapper">
          <QuizViewer :quiz="currentSlide.quiz" />
        </div>
        <div v-if="currentSlide.reviewQuiz" class="review-wrapper">
          <ReviewQuiz :config="currentSlide.reviewQuiz" />
        </div>
      </div>

      <!-- Feature Slide -->
      <div v-else-if="currentSlide.type === 'feature'" class="slide slide-feature">
        <div class="slide-header">
          <span class="slide-icon">{{ currentSlide.icon }}</span>
          <h2>{{ currentSlide.title }}</h2>
        </div>
        <div class="features-grid">
          <div v-for="(feature, index) in currentSlide.features" :key="index" class="feature-card">
            <div class="feature-icon">{{ feature.icon }}</div>
            <h3>{{ feature.title }}</h3>
            <p>{{ feature.description }}</p>
          </div>
        </div>
        <div v-if="currentSlide.quiz" class="quiz-wrapper">
          <QuizViewer :quiz="currentSlide.quiz" />
        </div>
        <div v-if="currentSlide.reviewQuiz" class="review-wrapper">
          <ReviewQuiz :config="currentSlide.reviewQuiz" />
        </div>
      </div>

      <!-- Summary Slide -->
      <div v-else-if="currentSlide.type === 'summary'" class="slide slide-summary">
        <div class="slide-header">
          <span class="slide-icon">{{ currentSlide.icon }}</span>
          <h2>{{ currentSlide.title }}</h2>
        </div>
        <div class="summary-content">
          <ul class="summary-list">
            <li v-for="(point, index) in (currentSlide.keyPoints || currentSlide.points)" :key="index">{{ point }}</li>
          </ul>
        </div>
        <div v-if="currentSlide.quiz" class="quiz-wrapper">
          <QuizViewer :quiz="currentSlide.quiz" />
        </div>
        <div v-if="currentSlide.reviewQuiz" class="review-wrapper">
          <ReviewQuiz :config="currentSlide.reviewQuiz" />
        </div>
      </div>

      <!-- Chart Slide (Cycle/Process) -->
      <div v-else-if="currentSlide.type === 'chart'" class="slide slide-chart">
        <div class="slide-header">
          <span class="slide-icon">{{ currentSlide.icon }}</span>
          <h2>{{ currentSlide.title }}</h2>
        </div>
        <div class="cycle-chart">
          <div v-for="(item, index) in currentSlide.data" :key="index" 
               class="cycle-item" 
               :style="{ borderColor: item.color }">
            <div class="cycle-season" :style="{ background: item.color }">
              {{ item.season }}
            </div>
            <div class="cycle-months">{{ item.months }}</div>
            <div class="cycle-phase">{{ item.phase }}</div>
            <ul class="cycle-tasks">
              <li v-for="(task, idx) in item.tasks" :key="idx">{{ task }}</li>
            </ul>
            <div class="cycle-risk">
              <span class="risk-label">主要風險：</span>{{ item.risk }}
            </div>
          </div>
        </div>
        <div v-if="currentSlide.quiz" class="quiz-wrapper">
          <QuizViewer :quiz="currentSlide.quiz" />
        </div>
      </div>

      <!-- Comparison Slide -->
      <div v-else-if="currentSlide.type === 'comparison'" class="slide slide-comparison">
        <div class="slide-header">
          <span class="slide-icon">{{ currentSlide.icon }}</span>
          <h2>{{ currentSlide.title }}</h2>
        </div>
        <h3 v-if="currentSlide.comparisonData.title" class="comparison-title">
          {{ currentSlide.comparisonData.title }}
        </h3>
        <div class="comparison-grid">
          <div class="comparison-column" :style="{ borderColor: currentSlide.comparisonData.left.color }">
            <div class="comparison-header" :style="{ background: currentSlide.comparisonData.left.color }">
              <span v-if="currentSlide.comparisonData.left.icon" class="comparison-icon">
                {{ currentSlide.comparisonData.left.icon }}
              </span>
              {{ currentSlide.comparisonData.left.label }}
            </div>
            <ul class="comparison-list">
              <li v-for="(item, idx) in currentSlide.comparisonData.left.items" :key="idx">{{ item }}</li>
            </ul>
          </div>
          <div class="comparison-column" :style="{ borderColor: currentSlide.comparisonData.right.color }">
            <div class="comparison-header" :style="{ background: currentSlide.comparisonData.right.color }">
              <span v-if="currentSlide.comparisonData.right.icon" class="comparison-icon">
                {{ currentSlide.comparisonData.right.icon }}
              </span>
              {{ currentSlide.comparisonData.right.label }}
            </div>
            <ul class="comparison-list">
              <li v-for="(item, idx) in currentSlide.comparisonData.right.items" :key="idx">{{ item }}</li>
            </ul>
          </div>
        </div>
        <p v-if="currentSlide.summary" class="comparison-summary">{{ currentSlide.summary }}</p>
        <div v-if="currentSlide.regionComparison" class="region-comparison">
          <h4>各產區機械化程度</h4>
          <div class="region-bars">
            <div v-for="(region, idx) in currentSlide.regionComparison" :key="idx" class="region-bar">
              <div class="region-name">{{ region.region }}</div>
              <div class="region-progress">
                <div class="progress-fill" :style="{ width: region.mechanization }"></div>
                <span class="progress-label">{{ region.mechanization }}</span>
              </div>
              <div class="region-reason">{{ region.reason }}</div>
            </div>
          </div>
        </div>
        <div v-if="currentSlide.quiz" class="quiz-wrapper">
          <QuizViewer :quiz="currentSlide.quiz" />
        </div>
      </div>

      <!-- Table Slide -->
      <div v-else-if="currentSlide.type === 'table'" class="slide slide-table">
        <div class="slide-header">
          <span class="slide-icon">{{ currentSlide.icon }}</span>
          <h2>{{ currentSlide.title }}</h2>
        </div>
        <div class="table-container">
          <table class="data-table">
            <thead>
              <tr>
                <th v-for="(header, idx) in currentSlide.tableData.headers" :key="idx">{{ header }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(row, idx) in currentSlide.tableData.rows" :key="idx">
                <td>{{ row.disease }}</td>
                <td>{{ row.condition }}</td>
                <td>{{ row.symptom }}</td>
                <td>{{ row.prevention }}</td>
                <td><span :class="['risk-badge', getRiskClass(row.risk)]">{{ row.risk }}</span></td>
              </tr>
            </tbody>
          </table>
        </div>
        <p v-if="currentSlide.note" class="table-note">💡 {{ currentSlide.note }}</p>
        <div v-if="currentSlide.quiz" class="quiz-wrapper">
          <QuizViewer :quiz="currentSlide.quiz" />
        </div>
      </div>

      <!-- Infographic Slide -->
      <div v-else-if="currentSlide.type === 'infographic'" class="slide slide-infographic">
        <div class="slide-header">
          <span class="slide-icon">{{ currentSlide.icon }}</span>
          <h2>{{ currentSlide.title }}</h2>
        </div>
        <p class="infographic-context">{{ currentSlide.infographicData.context }}</p>
        <div class="infographic-challenge">
          <h3>{{ currentSlide.infographicData.challenge.title }}</h3>
          <ul>
            <li v-for="(item, idx) in currentSlide.infographicData.challenge.items" :key="idx">{{ item }}</li>
          </ul>
        </div>
        <div class="infographic-solutions">
          <h3>防護措施對比</h3>
          <div class="solution-cards">
            <div v-for="(solution, idx) in currentSlide.infographicData.solutions" :key="idx" class="solution-card">
              <div class="solution-method">{{ solution.method }}</div>
              <div class="solution-stats">
                <div class="solution-stat">
                  <span class="stat-label">成本</span>
                  <span class="stat-value">{{ solution.cost }}</span>
                </div>
                <div class="solution-stat">
                  <span class="stat-label">有效性</span>
                  <span class="stat-value">{{ solution.effectiveness }}</span>
                </div>
                <div class="solution-stat">
                  <span class="stat-label">覆蓋範圍</span>
                  <span class="stat-value">{{ solution.coverage }}</span>
                </div>
              </div>
              <div class="solution-detail">{{ solution.detail }}</div>
            </div>
          </div>
        </div>
        <div class="infographic-result">✓ {{ currentSlide.infographicData.result }}</div>
        <div v-if="currentSlide.quiz" class="quiz-wrapper">
          <QuizViewer :quiz="currentSlide.quiz" />
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { computed } from 'vue'
import QuizViewer from './QuizViewer.vue'
import ReviewQuiz from './ReviewQuiz.vue'

const props = defineProps({
  slides: {
    type: Array,
    default: () => []
  },
  currentIndex: {
    type: Number,
    default: 0
  }
})

const currentSlide = computed(() => props.slides[props.currentIndex] || null)

const getRiskClass = (risk) => {
  if (risk.includes('極高')) return 'risk-critical'
  if (risk.includes('高')) return 'risk-high'
  if (risk.includes('中')) return 'risk-medium'
  return 'risk-low'
}
</script>

<style scoped>
.slide-viewer {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.slide-container {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
  overflow-y: auto;
}

.slide {
  width: 100%;
  max-width: 100%;
  background: white;
  border-radius: 0;
  padding: 60px 100px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
  animation: slideIn 0.4s ease-out;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(30px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

/* Cover Slide */
.slide-cover {
  text-align: center;
  color: white;
  min-height: 500px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.slide-cover-content {
  width: 100%;
}

.slide-cover .slide-icon {
  font-size: 80px;
  margin-bottom: 30px;
  display: block;
}

.slide-cover .slide-title {
  font-size: 48px;
  font-weight: 700;
  margin: 0 0 20px 0;
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.slide-cover .slide-subtitle {
  font-size: 24px;
  font-weight: 400;
  opacity: 0.95;
  margin: 0;
}

/* Slide Header */
.slide-header {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 40px;
  padding-bottom: 20px;
  border-bottom: 3px solid #667eea;
}

.slide-header .slide-icon {
  font-size: 48px;
}

.slide-header h2 {
  margin: 0;
  font-size: 36px;
  color: #2d3748;
  font-weight: 700;
}

.timeline-subtitle {
  margin: 8px 0 0 64px;
  font-size: 18px;
  color: #718096;
  font-weight: 400;
}

/* Content Slide */
.slide-body {
  font-size: 20px;
  line-height: 1.8;
  color: #4a5568;
}

.text-content {
  margin: 0 0 24px 0;
}

/* Stats Grid */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 24px;
  margin-top: 30px;
}

.stat-card {
  background: #f7fafc;
  padding: 28px;
  border-radius: 16px;
  border: 2px solid #e2e8f0;
  transition: all 0.3s;
}

.stat-card:hover {
  border-color: #667eea;
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.2);
}

.stat-icon {
  font-size: 40px;
  margin-bottom: 12px;
}

.stat-label {
  font-size: 16px;
  color: #718096;
  font-weight: 600;
  margin-bottom: 8px;
}

.stat-value {
  font-size: 18px;
  color: #2d3748;
  font-weight: 600;
  line-height: 1.5;
}

/* Timeline */
.timeline {
  position: relative;
  padding-left: 40px;
}

.timeline::before {
  content: '';
  position: absolute;
  left: 15px;
  top: 0;
  bottom: 0;
  width: 3px;
  background: linear-gradient(180deg, #667eea 0%, #764ba2 100%);
}

.timeline-item {
  position: relative;
  margin-bottom: 32px;
  padding-left: 32px;
}

.timeline-marker {
  position: absolute;
  left: 0;
  top: 8px;
  width: 16px;
  height: 16px;
  background: #667eea;
  border-radius: 50%;
  border: 3px solid white;
  box-shadow: 0 0 0 3px #667eea;
}

.timeline-year {
  font-size: 18px;
  font-weight: 700;
  color: #667eea;
  margin-bottom: 8px;
}

.timeline-text {
  font-size: 18px;
  color: #4a5568;
  line-height: 1.6;
}

/* Features Grid */
.features-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 28px;
}

.feature-card {
  background: #f7fafc;
  padding: 32px;
  border-radius: 16px;
  border: 2px solid #e2e8f0;
  transition: all 0.3s;
}

.feature-card:hover {
  border-color: #667eea;
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.2);
}

.feature-icon {
  font-size: 48px;
  margin-bottom: 16px;
}

.feature-card h3 {
  font-size: 22px;
  color: #2d3748;
  margin: 0 0 12px 0;
  font-weight: 700;
}

.feature-card p {
  font-size: 16px;
  color: #4a5568;
  line-height: 1.7;
  margin: 0;
}

/* Summary */
.summary-content {
  font-size: 20px;
}

.summary-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.summary-list li {
  position: relative;
  padding: 20px 20px 20px 60px;
  margin-bottom: 16px;
  background: #f7fafc;
  border-radius: 12px;
  border-left: 4px solid #667eea;
  line-height: 1.6;
  color: #2d3748;
}

.summary-list li::before {
  content: '✓';
  position: absolute;
  left: 20px;
  top: 50%;
  transform: translateY(-50%);
  width: 28px;
  height: 28px;
  background: #48bb78;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 16px;
}

/* Quiz wrapper spacing */
.quiz-wrapper {
  margin-top: 24px;
}

.review-wrapper {
  margin-top: 20px;
}

/* Cycle Chart */
.cycle-chart {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 24px;
  margin-top: 20px;
}

.cycle-item {
  background: #f7fafc;
  border: 3px solid;
  border-radius: 16px;
  padding: 24px;
  transition: all 0.3s;
}

.cycle-item:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
}

.cycle-season {
  color: white;
  font-size: 20px;
  font-weight: 700;
  padding: 8px 16px;
  border-radius: 8px;
  text-align: center;
  margin-bottom: 12px;
}

.cycle-months {
  font-size: 16px;
  color: #667eea;
  font-weight: 600;
  margin-bottom: 8px;
}

.cycle-phase {
  font-size: 18px;
  color: #2d3748;
  font-weight: 600;
  margin-bottom: 16px;
}

.cycle-tasks {
  list-style: none;
  padding: 0;
  margin: 0 0 16px 0;
}

.cycle-tasks li {
  font-size: 15px;
  color: #4a5568;
  padding: 6px 0 6px 20px;
  position: relative;
  line-height: 1.5;
}

.cycle-tasks li::before {
  content: '•';
  position: absolute;
  left: 0;
  color: #667eea;
  font-weight: bold;
}

.cycle-risk {
  background: #fff5f5;
  border-left: 3px solid #fc8181;
  padding: 8px 12px;
  border-radius: 4px;
  font-size: 14px;
}

.risk-label {
  font-weight: 600;
  color: #c53030;
  margin-right: 4px;
}

/* Comparison Grid */
.comparison-title {
  text-align: center;
  color: #2d3748;
  font-size: 24px;
  margin: 0 0 24px 0;
}

.comparison-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 24px;
  margin-bottom: 24px;
}

.comparison-column {
  background: #f7fafc;
  border: 3px solid;
  border-radius: 16px;
  overflow: hidden;
}

.comparison-header {
  color: white;
  padding: 16px;
  font-size: 20px;
  font-weight: 700;
  text-align: center;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
}

.comparison-icon {
  font-size: 24px;
}

.comparison-list {
  list-style: none;
  padding: 20px;
  margin: 0;
}

.comparison-list li {
  padding: 10px 0 10px 28px;
  position: relative;
  font-size: 16px;
  color: #2d3748;
  line-height: 1.6;
  border-bottom: 1px solid #e2e8f0;
}

.comparison-list li:last-child {
  border-bottom: none;
}

.comparison-list li::before {
  content: '✓';
  position: absolute;
  left: 0;
  color: #48bb78;
  font-weight: bold;
  font-size: 18px;
}

.comparison-summary {
  background: #e6fffa;
  border-left: 4px solid #319795;
  padding: 16px;
  border-radius: 8px;
  font-size: 16px;
  color: #2d3748;
  line-height: 1.6;
  margin-top: 20px;
}

.region-comparison {
  margin-top: 32px;
  background: #f7fafc;
  padding: 24px;
  border-radius: 12px;
}

.region-comparison h4 {
  color: #2d3748;
  font-size: 20px;
  margin: 0 0 20px 0;
}

.region-bars {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.region-bar {
  display: grid;
  grid-template-columns: 150px 1fr 200px;
  gap: 16px;
  align-items: center;
}

.region-name {
  font-weight: 600;
  color: #2d3748;
}

.region-progress {
  position: relative;
  background: #e2e8f0;
  height: 32px;
  border-radius: 16px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  transition: width 0.6s ease;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  padding-right: 12px;
}

.progress-label {
  position: absolute;
  right: 12px;
  top: 50%;
  transform: translateY(-50%);
  font-weight: 600;
  color: #2d3748;
  font-size: 14px;
}

.region-reason {
  font-size: 14px;
  color: #718096;
}

/* Table */
.table-container {
  overflow-x: auto;
  margin: 20px 0;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.data-table th {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 16px;
  text-align: left;
  font-weight: 600;
  font-size: 15px;
}

.data-table td {
  padding: 14px 16px;
  border-bottom: 1px solid #e2e8f0;
  color: #2d3748;
  font-size: 14px;
  line-height: 1.5;
}

.data-table tr:last-child td {
  border-bottom: none;
}

.data-table tr:hover {
  background: #f7fafc;
}

.risk-badge {
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 13px;
  font-weight: 600;
  display: inline-block;
}

.risk-critical {
  background: #fed7d7;
  color: #c53030;
}

.risk-high {
  background: #feebc8;
  color: #c05621;
}

.risk-medium {
  background: #fefcbf;
  color: #975a16;
}

.risk-low {
  background: #c6f6d5;
  color: #2f855a;
}

.table-note {
  background: #ebf8ff;
  border-left: 4px solid #3182ce;
  padding: 16px;
  border-radius: 8px;
  font-size: 15px;
  color: #2d3748;
  margin-top: 16px;
}

/* Infographic */
.infographic-context {
  background: #edf2f7;
  padding: 16px;
  border-radius: 8px;
  font-size: 16px;
  color: #2d3748;
  margin-bottom: 24px;
  border-left: 4px solid #667eea;
}

.infographic-challenge {
  background: #fff5f5;
  padding: 20px;
  border-radius: 12px;
  margin-bottom: 24px;
  border: 2px solid #fc8181;
}

.infographic-challenge h3 {
  color: #c53030;
  font-size: 20px;
  margin: 0 0 12px 0;
}

.infographic-challenge ul {
  margin: 0;
  padding-left: 24px;
}

.infographic-challenge li {
  color: #2d3748;
  margin: 8px 0;
  line-height: 1.6;
}

.infographic-solutions h3 {
  color: #2d3748;
  font-size: 22px;
  margin: 0 0 20px 0;
}

.solution-cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 16px;
  margin-bottom: 24px;
}

.solution-card {
  background: linear-gradient(135deg, #f7fafc 0%, #edf2f7 100%);
  border: 2px solid #cbd5e0;
  border-radius: 12px;
  padding: 20px;
  transition: all 0.3s;
}

.solution-card:hover {
  border-color: #667eea;
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.2);
}

.solution-method {
  font-size: 18px;
  font-weight: 700;
  color: #667eea;
  margin-bottom: 16px;
  text-align: center;
}

.solution-stats {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
  margin-bottom: 16px;
}

.solution-stat {
  background: white;
  padding: 10px;
  border-radius: 8px;
  text-align: center;
  border: 1px solid #e2e8f0;
}

.solution-stat .stat-label {
  font-size: 12px;
  color: #718096;
  display: block;
  margin-bottom: 4px;
}

.solution-stat .stat-value {
  font-size: 14px;
  font-weight: 600;
  color: #2d3748;
}

.solution-detail {
  font-size: 14px;
  color: #4a5568;
  line-height: 1.5;
  padding-top: 12px;
  border-top: 1px solid #cbd5e0;
}

.infographic-result {
  background: linear-gradient(135deg, #c6f6d5 0%, #9ae6b4 100%);
  padding: 20px;
  border-radius: 12px;
  font-size: 18px;
  font-weight: 600;
  color: #22543d;
  text-align: center;
  border: 2px solid #48bb78;
}

@media (max-width: 768px) {
  .slide-container {
    padding: 20px;
  }

  .slide {
    padding: 32px 24px;
  }

  .slide-cover .slide-title {
    font-size: 32px;
  }

  .slide-cover .slide-subtitle {
    font-size: 18px;
  }

  .slide-header h2 {
    font-size: 28px;
  }

  .stats-grid {
    grid-template-columns: 1fr;
  }

  .features-grid {
    grid-template-columns: 1fr;
  }

  .cycle-chart {
    grid-template-columns: 1fr;
  }

  .comparison-grid {
    grid-template-columns: 1fr;
  }

  .region-bar {
    grid-template-columns: 1fr;
  }

  .solution-cards {
    grid-template-columns: 1fr;
  }

  .solution-stats {
    grid-template-columns: 1fr;
  }
}
</style>
