<template>
  <section class="map-section">
    <div class="map-header">
      <h1>紐西蘭葡萄酒產區地圖</h1>
      <button class="back-to-course-btn" @click="backToCourse">← 返回課程</button>
    </div>
    <div class="map-info-bar" v-if="activeAOC.aoc">
      <div class="info-header-bar">
        <span class="aoc-info-title">
          <span class="aoc-dot" :style="{background: aocColor(activeAOC.group)}"></span>
          {{ activeAOC.aoc.replace('.geojson','').replace(/_/g,' ') }}
        </span>
        <div class="map-buttons">
          <button class="btn-reset" @click="resetMap">重置地圖</button>
        </div>
      </div>
      
      <div v-if="regionInfo" class="region-info-content">
        <div class="info-header">
          <div>
            <b>{{ regionInfo.name }}</b> 
            <span class="region-type">({{ regionInfo.type }})</span>
            <span v-if="regionInfo.hectare" class="region-hectare"> - {{ regionInfo.hectare }} 公頃</span>
          </div>
          <div class="style-badges">
            <div v-for="style in Array.isArray(regionInfo.style) ? regionInfo.style : [regionInfo.style]" 
                 :key="style" 
                 class="style-badge"
                 :style="{backgroundColor: styleColors[style] || '#999', color: ['白酒', '甜酒'].includes(style) ? '#333' : '#fff'}">
              {{ style }}
            </div>
          </div>
        </div>

        <div class="description">{{ regionInfo.description }}</div>

        <div v-if="regionInfo.details" class="details-section">
          <div v-if="regionInfo.details.introduction" class="detail-item">
            <div class="detail-title">產區介紹:</div>
            <p>{{ regionInfo.details.introduction }}</p>
          </div>
          <div v-if="regionInfo.details.climate" class="detail-item">
            <div class="detail-title">氣候:</div>
            <p>{{ regionInfo.details.climate }}</p>
          </div>
          <div v-if="regionInfo.details.subregions && Object.keys(regionInfo.details.subregions).length > 0" class="detail-item">
            <div class="detail-title">子產區:</div>
            <ul>
              <li v-for="(desc, subregion) in regionInfo.details.subregions" :key="subregion">
                <strong>{{ subregion }}:</strong> {{ desc }}
              </li>
            </ul>
          </div>
        </div>

        <div v-if="regionInfo.grapes" class="grape-section">
          <div class="grape-title">主要葡萄品種:</div>
          <div class="grape-badges">
            <div v-for="grape in Array.isArray(regionInfo.grapes) ? regionInfo.grapes : regionInfo.grapes.split(',').map(g => g.trim())"
                 :key="grape"
                 class="grape-badge"
                 :style="grapeBadgeStyle(grape)">
              {{ grape }}
            </div>
          </div>
        </div>
      </div>
      <div v-else class="no-info">無詳細產區資料</div>
    </div>
    <div ref="mapContainer" class="map"></div>
    <button class="btn-3d" @click="toggle3D">
      {{ is3D ? '2D' : '3D' }}
    </button>
    <div v-if="mapError" class="map-error">
      {{ mapError }}
    </div>
    <div v-if="isLoading" class="loading-overlay">
      <div class="loading-spinner"></div>
    </div>
  </section>
</template>

<script setup>
// 葡萄品種分類顏色
const grapeTypeColors = {
  red: '#8B0000',
  white: '#F0E68C',
  aromatic: '#87CEEB',
  other: '#bbb'
}

const redGrapes = [
  'Pinot Noir', 'Merlot', 'Syrah', 'Cabernet Sauvignon', 'Malbec', 'Pinotage', 'Chambourcin', 'Petit Verdot', 'Cabernet Franc', 'Montepulciano'
]
const whiteGrapes = [
  'Chardonnay', 'Sauvignon Blanc', 'Pinot Gris', 'Viognier', 'Grüner Veltliner', 'Riesling', 'Gewurztraminer'
]
const aromaticGrapes = [
  'Arneis'
]

function grapeBadgeStyle(grape) {
  if (redGrapes.includes(grape)) {
    return { backgroundColor: grapeTypeColors.red, color: '#fff', fontWeight: 'bold' }
  } else if (whiteGrapes.includes(grape)) {
    return { backgroundColor: grapeTypeColors.white, color: '#333', fontWeight: 'bold' }
  } else if (aromaticGrapes.includes(grape)) {
    return { backgroundColor: grapeTypeColors.aromatic, color: '#333', fontWeight: 'bold' }
  } else {
    return { backgroundColor: grapeTypeColors.other, color: '#333' }
  }
}
import { ref, onMounted, onUnmounted, nextTick, watch } from 'vue'
import mapboxgl from 'mapbox-gl'
import 'mapbox-gl/dist/mapbox-gl.css'
import * as turf from '@turf/turf'

const props = defineProps({
  activeAOC: Object,
  regionInfo: Object,
  styleColors: Object,
})

const emit = defineEmits(['resetMap', 'back-to-course'])

const backToCourse = () => {
  emit('back-to-course')
}

const isLoading = ref(false)
const mapError = ref(null)
const mapContainer = ref(null)
let map = null
const is3D = ref(false)
const geojsonCache = new Map()

function aocColor(groupName) {
  return '#006400' // DarkGreen for New Zealand
}

const styleColors = {
  '紅酒': '#8B0000',
  '白酒': '#F0E68C',
  '甜酒': '#FFD700',
  '氣泡酒': '#87CEEB',
  '粉紅酒': '#FFB6C1'
}

const showAOCGeojson = async (groupName, aocFile) => {
  if (!map) return

  const geojsonPath = `/geojson/NewZealand/${aocFile}`

  isLoading.value = true
  mapError.value = null

  try {
    let geojson
    if (geojsonCache.has(geojsonPath)) {
      geojson = geojsonCache.get(geojsonPath)
    } else {
      const res = await fetch(geojsonPath)
      if (!res.ok) throw new Error(`無法載入 geojson (${res.status})`)
      geojson = await res.json()
      geojsonCache.set(geojsonPath, geojson)
    }

    if (map.getLayer('aoc-fill')) map.removeLayer('aoc-fill')
    if (map.getLayer('aoc-outline')) map.removeLayer('aoc-outline')
    if (map.getSource('aoc')) map.removeSource('aoc')

    map.addSource('aoc', { type: 'geojson', data: geojson })
    
    map.addLayer({
      id: 'aoc-fill',
      type: 'fill',
      source: 'aoc',
      paint: {
        'fill-color': aocColor(groupName),
        'fill-opacity': 0.3
      }
    })
    map.addLayer({
      id: 'aoc-outline',
      type: 'line',
      source: 'aoc',
      paint: {
        'line-color': '#fff',
        'line-width': 2
      }
    })

    const bbox = turf.bbox(geojson)
    map.fitBounds(bbox, { padding: 40, duration: 800 })

  } catch (err) {
    console.error('載入 geojson 失敗:', err)
    mapError.value = `載入 geojson 失敗: ${err.message}`
  } finally {
    isLoading.value = false
  }
}

const resetMap = () => {
  emit('resetMap')
}

const toggle3D = () => {
  is3D.value = !is3D.value
  if (map) {
    map.easeTo({ pitch: is3D.value ? 45 : 0, duration: 800 })
  }
}

const initMap = async (retry = 0) => {
  try {
    if (!mapContainer.value) {
      if (retry < 5) {
        setTimeout(() => initMap(retry + 1), 200)
      } else {
        mapError.value = '無法獲取地圖容器'
      }
      return
    }
    
    mapboxgl.accessToken = 'pk.eyJ1IjoiY2h1bmdzaHVsZWUiLCJhIjoiY21kcTZqbHU1MDNnODJsczZ5NXBtNms2NCJ9.UThWFp3_47qETAMZWhdrTg'
    
    map = new mapboxgl.Map({
      container: mapContainer.value,
      style: 'mapbox://styles/mapbox/satellite-streets-v12',
      center: [174.886, -40.9006],
      zoom: 4,
      pitch: is3D.value ? 45 : 0,
      bearing: 0
    })
    
    map.on('load', async () => {
      map.addControl(new mapboxgl.NavigationControl(), 'top-right')
      map.addControl(new mapboxgl.FullscreenControl(), 'top-right')
    })
    
    map.on('error', (err) => {
      console.error('地圖錯誤:', err)
      mapError.value = `地圖錯誤: ${err.error?.message || '未知錯誤'}`
    })
    
    mapError.value = null
  } catch (err) {
    console.error('地圖初始化錯誤:', err)
    mapError.value = `初始化錯誤: ${err.message}`
  }
}

watch(() => props.activeAOC, (newAOC, oldAOC) => {
  if (newAOC && newAOC.aoc) {
    if (newAOC.aoc !== oldAOC?.aoc) {
      showAOCGeojson(newAOC.group, newAOC.aoc)
    }
  } else if (map.getLayer('aoc-fill')) {
    map.removeLayer('aoc-fill')
    map.removeLayer('aoc-outline')
    map.removeSource('aoc')
    map.flyTo({ center: [174.886, -40.9006], zoom: 4 })
  }
}, { deep: true })

onMounted(async () => {
  await nextTick()
  setTimeout(async () => {
    await initMap()
  }, 100)
})

onUnmounted(() => {
  if (map) {
    map.remove()
    map = null
  }
})
</script>

<style scoped>
.map-section {
  flex: 1;
  position: relative;
  height: 100%;
  overflow: hidden;
}

.map {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.map-header {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  background: rgba(255, 255, 255, 0.85);
  padding: 8px 20px;
  z-index: 10;
  text-align: center;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.map-header h1 {
  margin: 0;
  font-size: 1.5rem;
  color: #006400; /* DarkGreen for NZ */
}

.back-to-course-btn {
  position: absolute;
  right: 20px;
  top: 50%;
  transform: translateY(-50%);
  padding: 8px 20px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 600;
  transition: all 0.2s;
  box-shadow: 0 2px 8px rgba(102, 126, 234, 0.3);
}

.back-to-course-btn:hover {
  transform: translateY(-50%) translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.map-info-bar {
  position: absolute;
  bottom: 20px;
  left: 20px;
  background: white;
  padding: 15px;
  border-radius: 8px;
  max-width: 450px;
  max-height: 25vh;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  z-index: 10;
  display: flex;
  flex-direction: column;
}

.info-header-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-shrink: 0;
}

.aoc-info-title {
  display: flex;
  align-items: center;
  font-size: 1.1rem;
  font-weight: bold;
}

.aoc-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  margin-right: 8px;
  flex-shrink: 0;
}

.map-buttons {
  display: flex;
  gap: 8px;
}

.btn-reset {
  padding: 6px 12px;
  background: #f44336;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.8rem;
}

.btn-reset:hover {
  background: #d32f2f;
}

.region-info-content {
  margin-top: 12px;
  font-size: 0.95rem;
  line-height: 1.5;
  text-align: left;
  overflow-y: auto;
  padding-right: 10px;
}

.info-header {
  margin-bottom: 10px;
}

.region-type, .region-hectare {
  font-size: 0.9rem;
  color: #666;
}

.style-badges {
  display: flex;
  gap: 5px;
  flex-wrap: wrap;
  margin-top: 8px;
}

.style-badge {
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 0.8rem;
  font-weight: bold;
  display: inline-block;
  text-align: center;
  box-shadow: 0 1px 3px rgba(0,0,0,0.2);
}

.description {
  margin-top: 10px;
  font-size: 0.9rem;
  text-align: left;
}

.details-section {
  margin-top: 15px;
  border-top: 1px solid #eee;
  padding-top: 15px;
}

.detail-item {
  margin-bottom: 10px;
}

.detail-title {
  font-weight: bold;
  font-size: 1rem;
  color: #333;
  margin-bottom: 5px;
}

.detail-item p, .detail-item ul {
  margin: 0;
  padding-left: 5px;
  font-size: 0.9rem;
}

.detail-item ul {
  padding-left: 20px;
}

.detail-item li {
  margin-bottom: 5px;
}

.grape-section {
  margin: 15px 0;
  border-top: 1px solid #eee;
  padding-top: 15px;
}

.grape-title {
  font-weight: bold;
  font-size: 1rem;
  color: #333;
  margin-bottom: 10px;
}

.grape-badges {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.grape-badge {
  padding: 4px 10px;
  border-radius: 15px;
  font-size: 0.8rem;
  white-space: nowrap;
  font-weight: bold;
  box-shadow: 0 1px 2px rgba(0,0,0,0.08);
  border: 1px solid #ddd;
}

.no-info {
  margin-top: 10px;
  color: #888;
  font-size: 0.9rem;
  font-style: italic;
}

.btn-3d {
  position: absolute;
  top: 80px;
  right: 10px;
  padding: 8px 15px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  z-index: 10;
  font-weight: bold;
}

.btn-3d:hover {
  background: #388E3C;
}

.loading-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(255, 255, 255, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 20;
}

.loading-spinner {
  width: 50px;
  height: 50px;
  border: 5px solid #f3f3f3;
  border-top: 5px solid #3498db;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.map-error {
  position: absolute;
  top: 70px;
  left: 50%;
  transform: translateX(-50%);
  background: #f44336;
  color: white;
  padding: 10px 20px;
  border-radius: 4px;
  z-index: 30;
  max-width: 80%;
  text-align: center;
}

@media (max-width: 768px) {
  .map-info-bar {
    max-width: calc(100% - 40px);
    width: auto;
    bottom: 10px;
    left: 10px;
    max-height: 30vh;
  }
  
  .map-header {
    padding: 8px 10px;
  }
  
  .map-header h1 {
    font-size: 1.1rem;
    padding-right: 120px;
  }
  
  .back-to-course-btn {
    right: 10px;
    padding: 6px 14px;
    font-size: 13px;
  }
}
</style>