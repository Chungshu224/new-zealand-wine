<template>
  <div class="main-layout">
    <AOCList
      v-model:search="search"
      :filteredGroups="filteredGroups"
      :expandedGroups="expandedGroups"
      :toggleGroup="toggleGroup"
      :activeAOC="activeAOC"
      :aocColor="aocColor"
      @selectAOC="showAOCGeojson"
    />
    
    <MapSection
      :activeAOC="activeAOC"
      :regionInfo="regionInfo"
      :styleColors="styleColors"
      @resetMap="resetMap"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import AOCList from './AOCList.vue'
import MapSection from './MapSection.vue'

// 狀態管理
const search = ref('')
const activeAOC = ref({ group: 'Regional', aoc: 'Bordeaux_AOC.geojson' })
const regionInfo = ref(null)
const regionsData = ref([])

// 展開/收合狀態
const expandedGroups = ref({
  'Regional': true,
  'LeftBank-Medoc': false,
  'LeftBank-Graves': false,
  'RightBank-Libournais': false,
  'RightBank-Blaye': false,
  'Entre-Deux-Mers': false,
  'Sauternais': false
})

const toggleGroup = (groupName) => {
  expandedGroups.value[groupName] = !expandedGroups.value[groupName]
}

// 色彩對應
function aocColor(groupName) {
  if (groupName.includes('LeftBank')) return '#8B0000'
  if (groupName.includes('RightBank')) return '#4169E1'
  if (groupName === 'Sauternais') return '#FFD700'
  if (groupName === 'Regional') return '#8B5C2A'
  if (groupName === 'Entre-Deux-Mers') return '#2E8B57'
  return '#aaa'
}

// 搜尋過濾
const aocGroups = ref({
  'Regional': [
    'Bordeaux_AOC.geojson',
    'Bordeaux-Superior_AOC.geojson',
    'Cotes-de-Bordeaux_AOC.geojson',
    'Cremant-de-Bordeaux_AOC.geojson'
  ],
  'LeftBank-Medoc': [
    'Medoc_AOC.geojson',
    'Haut-Medoc_AOC.geojson',
    'Margaux_AOC.geojson',
    'Pauillac_AOC.geojson',
    'St-Julien_AOC.geojson',
    'St-Estephe_AOC.geojson',
    'Listrac-Medoc_AOC.geojson',
    'Moulis-en-Medoc_AOC.geojson'
  ],
  'LeftBank-Graves': [
    'Graves_AOC.geojson',
    'Pessac-Leognan_AOC.geojson',
    'Graves-Superieures_AOC.geojson'
  ],
  'RightBank-Libournais': [
    'Pomerol_AOC.geojson',
    'St-Emilion_AOC.geojson',
    'St-Emilion-Grand-Cru_AOC.geojson',
    'Fronsac_AOC.geojson',
    'Canon-Fronsac_AOC.geojson',
    'Lalande-de-Pomerol_AOC.geojson',
    'Lussac-St-Emilion_AOC.geojson',
    'Montagne-St-Emilion_AOC.geojson',
    'Puisseguin-St-Emilion_AOC.geojson',
    'St-Georges-St-Emilion_AOC.geojson',
    'Castillon-Cotes-de-Bordeaux_AOC.geojson'
  ],
  'RightBank-Blaye': [
    'Blaye_AOC.geojson',
    'Cotes-de-Bourg_AOC.geojson',
    'Côtes de Blaye_AOC.geojson',
    'Côtes-de-Bordeaux-Blaye_AOC.geojson',
    'Côtes-de-Bordeaux_AOC.geojson'
  ],
  'Entre-Deux-Mers': [
    'Entre-Deux-Mers_AOC.geojson',
    'Cadillac_AOC.geojson',
    'Loupiac_AOC.geojson',
    'Sainte-Croix-du-Mont_AOC.geojson',
    'Entre-deux-Mers-Haut-Benauge_AOC.geojson',
    'Graves-of-Vayres_AOC.geojson',
    'St-Foy-Bordeaux_AOC.geojson',
    'Bordeaux Haut-Benauge_AOC.geojson',
    'Côtes-de-Bordeaux-Cadillac_AOC.geojson',
    'Côtes-de-Bordeaux-Francs_AOC.geojson',
    '1er-Côtes-de-Bordeaux_AOC.geojson',
    'Cotes-de-Bordeaux-St-Macaire_AOC.geojson'
  ],
  'Sauternais': [
    'Sauternes_AOC.geojson',
    'Barsac_AOC.geojson',
    'Cerons_AOC.geojson'
  ]
})

const filteredGroups = computed(() => {
  if (!search.value) return aocGroups.value
  
  const result = {}
  for (const [group, aocs] of Object.entries(aocGroups.value)) {
    const filtered = aocs.filter(aoc => 
      aoc.toLowerCase().includes(search.value.toLowerCase())
    )
    if (filtered.length) result[group] = filtered
  }
  
  return result
})

watch(filteredGroups, (val) => {
  // 有搜尋結果時，自動展開該群組
  for (const key in expandedGroups.value) {
    expandedGroups.value[key] = !!val[key]
  }
})

// 葡萄酒風格顏色映射
const styleColors = {
  '紅酒': '#8B0000',    // 深紅色
  '白酒': '#F0E68C',    // 淡黃色
  '甜酒': '#FFD700',    // 金黃色
  '氣泡酒': '#87CEEB',  // 淡藍色
  '粉紅酒': '#FFB6C1'   // 粉紅色
}

// 顯示 AOC 地圖
const showAOCGeojson = async (groupName, aocFile) => {
  activeAOC.value = { group: groupName, aoc: aocFile }
  
  // 查找產區資訊
  const aocId = aocFile.replace('.geojson', '')
  regionInfo.value = Array.isArray(regionsData.value) ? 
    regionsData.value.find(r => r.id === aocId) || null : null
}

// 重置地圖
const resetMap = () => {
  showAOCGeojson('Regional', 'Bordeaux_AOC.geojson')
}

// 載入產區資訊
const loadRegionsData = async () => {
  try {
    const res = await fetch('/bordeaux-regions.json')
    if (res.ok) {
      regionsData.value = await res.json()
    } else {
      throw new Error(`HTTP ${res.status}`)
    }
  } catch (err) {
    console.error('載入產區資料失敗:', err)
    regionsData.value = []
  }
}

onMounted(async () => {
  // 載入產區資料
  await loadRegionsData()
  
  // 預設顯示波爾多整體
  await showAOCGeojson('Regional', 'Bordeaux_AOC.geojson')
})
</script>

<style>
/* 全局樣式 */
html, body {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  overflow: hidden;
}

#app {
  height: 100%;
  width: 100%;
}
</style>

<style scoped>
/* BordeauxMap.vue 內的樣式 */
.main-layout {
  display: flex;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}

/* 響應式設計 */
@media (max-width: 768px) {
  .main-layout {
    flex-direction: column;
    height: 100%;
    width: 100%;
  }
  
  :deep(.aoc-list) {
    height: 30%;
    width: 100%;
    overflow-y: auto;
    flex-shrink: 0;
  }
}
</style>


