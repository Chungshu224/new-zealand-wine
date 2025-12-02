<template>
  <div class="aoc-list">
    <h2>紐西蘭產區清單</h2>
    
    <input
      type="text"
      class="aoc-search"
      placeholder="搜尋產區..."
      v-model="searchModel"
    />
    
  <div v-for="(regions, island) in sortedIslands" :key="island" class="aoc-group island-group">
      <div class="group-header island-header" @click="toggleIsland(island)">
        <span class="group-icon">{{ expandedIslands[island] ? '▼' : '►' }}</span>
        <span class="group-name">{{ island }}</span>
      </div>
      
      <div v-show="expandedIslands[island]" class="group-content island-content">
        <div v-for="(aocs, region) in regions" :key="region" class="aoc-group region-group">
          <div class="group-header region-header" @click="toggleRegion(region)">
            <span class="group-icon">{{ expandedRegions[region] ? '▼' : '►' }}</span>
            <span class="group-name">{{ region }}</span>
          </div>

          <div v-show="expandedRegions[region]" class="group-content region-content">
            <div
              v-for="aoc in aocs"
              :key="aoc"
              class="aoc-item"
              :class="{ active: isActive(island, aoc) }"
              @click="$emit('selectAOC', island, aoc)"
            >
              <span class="aoc-dot" :style="{ background: aocColor(island) }"></span>
              <span>{{ formatAOCName(aoc) }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>


<script setup>
import { computed } from 'vue'


const props = defineProps({
  search: String,
  aocGroups: Object,
  expandedIslands: Object,
  expandedRegions: Object,
  toggleIsland: Function,
  toggleRegion: Function,
  activeAOC: Object,
  aocColor: Function
})

// 紐西蘭由北到南的主要產區順序
const regionOrder = [
  'Northland',
  'Auckland',
  'Kumeu',
  'Gisborne',
  "Hawke's Bay",
  'Wairarapa',
  'Nelson',
  'Marlborough',
  'North Canterbury',
  'Waitaki Valley North Otago',
  'Central Otago'
]

function sortRegions(regions) {
  // regions: { regionName: [...] }
  const ordered = {}
  regionOrder.forEach(region => {
    if (regions[region]) ordered[region] = regions[region]
  })
  // 其他未列名的產區放最後
  Object.keys(regions).forEach(region => {
    if (!ordered[region]) ordered[region] = regions[region]
  })
  return ordered
}

const sortedIslands = computed(() => {
  // aocGroups: { island: { region: [...] } }
  const result = {}
  Object.entries(props.aocGroups).forEach(([island, regions]) => {
    result[island] = sortRegions(regions)
  })
  return result
})

const emit = defineEmits(['update:search', 'selectAOC'])

const searchModel = computed({
  get: () => props.search,
  set: (val) => emit('update:search', val)
})

const formatAOCName = (aoc) => {
  return aoc
    .replace('.geojson', '')
    .replace(/_/g, ' ')
    .replace(/-/g, ' ')
}

const isActive = (group, aoc) => {
  return props.activeAOC?.group === group && props.activeAOC?.aoc === aoc
}
</script>

<style scoped>
.aoc-list {
  flex: 0 0 320px;
  height: 100%;
  overflow-y: auto;
  background: #f8f8f8;
  border-right: 1px solid #ddd;
  padding: 20px 15px;
  box-sizing: border-box;
}

h2 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #006400; /* DarkGreen for NZ */
  letter-spacing: 1px;
}

.aoc-search {
  width: 100%;
  padding: 8px 12px;
  border-radius: 4px;
  border: 1px solid #ccc;
  margin-bottom: 15px;
  font-size: 0.9rem;
}

.aoc-search:focus {
  outline: none;
  border-color: #006400;
}

.group-header {
  display: flex;
  align-items: center;
  padding: 8px 5px;
  cursor: pointer;
  border-radius: 4px;
}

.group-header:hover {
  background: rgba(0, 0, 0, 0.05);
}

.island-header .group-name {
  font-size: 1.1rem;
  font-weight: bold;
  color: #333;
}

.region-header .group-name {
  font-size: 1rem;
  font-weight: 500;
}

.group-icon {
  font-size: 0.7rem;
  margin-right: 8px;
  width: 10px;
}

.island-content {
  margin-left: 10px;
}

.region-group {
  margin-left: 10px;
}

.region-content {
  margin-left: 20px;
}

.aoc-item {
  display: flex;
  align-items: center;
  padding: 6px 8px;
  margin: 2px 0;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9rem;
}

.aoc-item:hover {
  background: rgba(0, 0, 0, 0.05);
}

.aoc-item.active {
  background: rgba(0, 100, 0, 0.1);
  font-weight: 500;
}

.aoc-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  margin-right: 8px;
  flex-shrink: 0;
}

@media (max-width: 768px) {
  .aoc-list {
    width: 100%;
    height: auto;
    flex: 0 0 auto;
    border-right: none;
    border-bottom: 1px solid #ddd;
  }
}
</style>