<!-- src/components/AppLayout.vue -->
<template>
  <div class="app-layout">
    <div
      class="sidebar"
      :style="isMobile ? { transform: sidebarOpen ? 'translateY(0)' : 'translateY(-100%)' } : {}"
    >
      <slot name="sidebar"></slot>
    </div>
    <div class="main-content">
      <slot name="main"></slot>
    </div>
    
    <!-- 行動裝置選單按鈕 -->
    <button v-if="isMobile" class="mobile-menu-btn" @click="toggleSidebar">
      {{ sidebarOpen ? '關閉選單' : '開啟選單' }}
    </button>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const isMobile = ref(window.innerWidth <= 768);
const sidebarOpen = ref(!isMobile.value);

const toggleSidebar = () => {
  sidebarOpen.value = !sidebarOpen.value;
};

const handleResize = () => {
  isMobile.value = window.innerWidth <= 768;
  if (!isMobile.value) {
    sidebarOpen.value = true;
  }
};

onMounted(() => {
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
});
</script>

<style scoped>
.app-layout {
  display: flex;
  height: 100vh;
  position: relative;
}

.sidebar {
  flex: 0 0 auto;
  width: 320px;
  max-width: 100%;
  height: 100%;
  overflow-y: auto;
  transition: all 0.3s ease;
}

.main-content {
  flex: 1;
  position: relative;
  height: 100%;
}

.mobile-menu-btn {
  display: none;
  position: fixed;
  left: 10px;
  top: 10px;
  z-index: 1000;
  padding: 12px 20px;
  background: #fff;
  border: 2px solid #8b4513;
  border-radius: 8px;
  font-size: 14px;
  font-weight: 600;
  color: #8b4513;
  cursor: pointer;
  min-height: 44px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
  transition: all 0.2s;
}

.mobile-menu-btn:hover {
  background: #8b4513;
  color: #fff;
}

.mobile-menu-btn:active {
  transform: scale(0.98);
}

@media (max-width: 768px) {
  .app-layout {
    flex-direction: column;
  }
  
  .sidebar {
    position: absolute;
    z-index: 100;
    /* transform is now handled via :style binding in the template */
    height: auto;
    max-height: 60vh;
    width: 100%;
    transform: translateY(v-bind(sidebarOpen ? '0' : '-100%'));
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  }
  
  .mobile-menu-btn {
    display: block;
  }

  .main-content {
    margin-top: 60px;
  }
}

@media (max-width: 480px) {
  .mobile-menu-btn {
    padding: 10px 16px;
    font-size: 13px;
    left: 8px;
    top: 8px;
    min-height: 40px;
  }

  .sidebar {
    max-height: 70vh;
  }
}
</style>