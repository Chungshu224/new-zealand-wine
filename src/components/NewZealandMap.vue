<template>
  <div class="main-layout">
    <AOCList
      v-model:search="search"
      :aocGroups="filteredAocGroups"
      :expandedIslands="expandedIslands"
      :expandedRegions="expandedRegions"
      :toggleIsland="toggleIsland"
      :toggleRegion="toggleRegion"
      :activeAOC="activeAOC"
      :aocColor="aocColor"
      @selectAOC="showAOCGeojson"
    />
    
    <MapSection
      :activeAOC="activeAOC"
      :regionInfo="regionInfo"
      :styleColors="styleColors"
      @resetMap="resetMap"
      @back-to-course="$emit('back-to-course')"
    />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import AOCList from './AOCList.vue'
import MapSection from './MapSection.vue'

// State
const props = defineProps({
  focusRegion: {
    type: String,
    default: ''
  }
})

const search = ref('')
const activeAOC = ref({ group: '', aoc: '' })
const regionInfo = ref(null)
const regionsData = ref([])

// Expansion states
const expandedIslands = ref({
  'North Island': true,
  'South Island': true,
})
const expandedRegions = ref({})

const toggleIsland = (islandName) => {
  expandedIslands.value[islandName] = !expandedIslands.value[islandName]
}

const toggleRegion = (regionName) => {
  expandedRegions.value[regionName] = !expandedRegions.value[regionName]
}

// Color mapping
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

// Data loading and processing
const aocFiles = [
    'Auckland.geojson', 'Bannockburn.geojson', 'Central_Hawkes_Bay.geojson',
    'Central_Otago.geojson', 'Gisborne.geojson', 'Gladstone.geojson',
    'Hawkes_Bay.geojson', 'Kumeu.geojson', 'Marlborough.geojson',
    'Martinborough.geojson', 'Matakana.geojson', 'Nelson.geojson',
    'North_Canterbury.geojson', 'Northland.geojson', 'Waiheke_Island.geojson',
    'Waipara_Valley.geojson', 'Waitaki_Valley_North_Otago.geojson'
];

const geojsonToRegionMap = {
    'Auckland.geojson': 'Auckland',
    'Waiheke_Island.geojson': 'Auckland',
    'Kumeu.geojson': 'Auckland',
    'Matakana.geojson': 'Auckland',
    'Bannockburn.geojson': 'CentralOtago',
    'Central_Otago.geojson': 'CentralOtago',
    'Central_Hawkes_Bay.geojson': 'HawkesBay',
    'Hawkes_Bay.geojson': 'HawkesBay',
    'Gisborne.geojson': 'Gisborne',
    'Gladstone.geojson': 'Wairarapa',
    'Martinborough.geojson': 'Wairarapa',
    'Marlborough.geojson': 'Marlborough',
    'Nelson.geojson': 'Nelson',
    'North_Canterbury.geojson': 'NorthCanterbury',
    'Waipara_Valley.geojson': 'NorthCanterbury',
    'Northland.geojson': 'Northland',
    'Waitaki_Valley_North_Otago.geojson': 'WaitakiValley'
};

const aocGroups = computed(() => {
    const groups = { 'North Island': {}, 'South Island': {} };
    if (!regionsData.value.length) {
        return groups;
    }

    const regionMap = new Map(regionsData.value.map(r => [r.id, r]));

    for (const file of aocFiles) {
        const regionId = geojsonToRegionMap[file];
        if (regionId) {
            const region = regionMap.get(regionId);
            if (region) {
                const island = region.island;
                if (island && groups[island]) {
                  if (!groups[island][region.name]) {
                      groups[island][region.name] = [];
                  }
                  groups[island][region.name].push(file);
                }
            }
        }
    }
    return groups;
});

const filteredAocGroups = computed(() => {
  if (!search.value) return aocGroups.value;
  const lowerCaseSearch = search.value.toLowerCase();
  const result = { 'North Island': {}, 'South Island': {} };

  for (const island in aocGroups.value) {
    for (const region in aocGroups.value[island]) {
      const aocs = aocGroups.value[island][region];
      const filteredAocs = aocs.filter(aoc => aoc.toLowerCase().replace('.geojson','').includes(lowerCaseSearch));

      if (filteredAocs.length > 0) {
        if (!result[island]) result[island] = {};
        result[island][region] = filteredAocs;
      }
    }
  }
  return result;
});

// Map interaction
const showAOCGeojson = async (groupName, aocFile) => {
  activeAOC.value = { group: groupName, aoc: aocFile }
  const aocId = aocFile.replace('.geojson', '').replace(/_/g, ' ');
  
  let foundRegion = null;
  for (const region of regionsData.value) {
    if (region.id.toLowerCase() === aocId.toLowerCase() || region.name.toLowerCase() === aocId.toLowerCase()) {
      foundRegion = region;
      break;
    }
    if (region.subregions && region.subregions.some(sub => sub.toLowerCase() === aocId.toLowerCase())) {
      foundRegion = region;
      break;
    }
  }
  regionInfo.value = foundRegion;
}

const resetMap = () => {
  activeAOC.value = { group: '', aoc: '' }
  regionInfo.value = null;
}

// Lifecycle
const loadRegionsData = async () => {
  try {
    const res = await fetch('/new-zealand-regions.json')
    if (res.ok) {
      regionsData.value = await res.json()
      // Initialize expandedRegions for all regions
      regionsData.value.forEach(r => { expandedRegions.value[r.name] = false; });
    } else {
      throw new Error(`HTTP ${res.status}`)
    }
  } catch (err) {
    console.error('載入產區資料失敗:', err)
    regionsData.value = []
  }
}

onMounted(async () => {
  await loadRegionsData()
})

// watch external focusRegion prop to highlight on map
watch(() => props.focusRegion, async (newRegion) => {
  if (!newRegion) return
  // wait for regions data
  if (!regionsData.value.length) await loadRegionsData()

  // find geojson file for region
  let targetFile = null
  for (const [file, regionId] of Object.entries(geojsonToRegionMap)) {
    if (regionId.toLowerCase() === newRegion.toLowerCase()) {
      targetFile = file
      break
    }
  }
  if (!targetFile) return

  // find region object to get island (group)
  const regionObj = regionsData.value.find(r => r.id.toLowerCase() === newRegion.toLowerCase())
  const groupName = regionObj ? regionObj.island : ''
  // call existing function to show on map
  await showAOCGeojson(groupName, targetFile)
})

</script>

<style>
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

@media (max-width: 768px) {
  .main-layout {
    flex-direction: column;
    height: 100%;
    width: 100%;
  }
  
  :deep(.aoc-list) {
    height: 40%;
    min-height: 300px;
    max-height: 50vh;
    width: 100%;
    overflow-y: auto;
    flex-shrink: 0;
    border-right: none;
    border-bottom: 3px solid #8b4513;
  }

  :deep(.search-box) {
    padding: 12px;
    font-size: 16px;
    min-height: 44px;
  }

  :deep(.region-item),
  :deep(.island-header) {
    padding: 12px 16px;
    min-height: 48px;
    font-size: 15px;
  }

  :deep(.aoc-item) {
    padding: 10px 16px;
    min-height: 44px;
  }
}

@media (max-width: 480px) {
  :deep(.aoc-list) {
    height: 35%;
    min-height: 250px;
  }

  :deep(.search-box) {
    padding: 10px;
    font-size: 15px;
  }

  :deep(.region-item),
  :deep(.island-header) {
    padding: 10px 12px;
    font-size: 14px;
  }
}
</style>