<script setup lang="ts">
import { onMounted, ref, onBeforeUnmount, computed } from 'vue';
import FEACanvas from './components/FEACanvas.vue';
import ElementInfo from './components/ElementInfo.vue';
import MeshControls from './components/MeshControls.vue';
import { useFEAStore } from './store/fea';

const store = useFEAStore();
const sidebarOpen = ref(true);

function getIsMobile() {
  return window.innerWidth < 768;
}

const isMobile = ref(getIsMobile());

function handleResize() {
  isMobile.value = getIsMobile();
}

onMounted(() => {
  store.loadPreset('cantilever');
  window.addEventListener('resize', handleResize);
});

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize);
});

function toggleSidebar() {
  sidebarOpen.value = !sidebarOpen.value;
}

const sidebarLabel = computed(() => {
  if (isMobile.value) return sidebarOpen.value ? '收起面板' : '展开面板';
  return sidebarOpen.value ? '收起侧边栏' : '展开侧边栏';
});
</script>

<template>
  <div class="h-screen bg-slate-950 text-slate-100 flex flex-col">
    <!-- Header -->
    <header class="bg-slate-900 border-b border-slate-800 px-3 md:px-6 py-3 flex items-center justify-between gap-2">
      <h1 class="text-sm md:text-lg font-bold text-purple-400 flex-shrink-0">
        🔬 有限元应力热力图
      </h1>
      <div class="flex items-center gap-3">
        <div class="text-[10px] md:text-xs text-slate-500 hidden sm:block">
          节点: {{ store.model.nodes.length }} |
          单元: {{ store.model.elements.length }}
        </div>
        <button
          @click="toggleSidebar"
          class="px-2 py-1 text-xs rounded bg-slate-800 text-slate-300 hover:bg-slate-700 transition flex-shrink-0"
          :title="sidebarLabel"
        >
          <span v-if="isMobile">{{ sidebarOpen ? '收起 ▲' : '面板 ▼' }}</span>
          <span v-else>{{ sidebarOpen ? '收起 ⟩' : '侧边栏 ⟨' }}</span>
        </button>
      </div>
    </header>

    <!-- Main content -->
    <div class="flex-1 overflow-hidden flex flex-col md:flex-row">
      <!-- Canvas area -->
      <div class="flex-1 p-2 md:p-3 min-h-0 flex flex-col">
        <FEACanvas />
      </div>

      <!-- Right sidebar -->
      <div
        class="bg-slate-900 border-slate-800 p-2 md:p-3 flex flex-col gap-2 md:gap-3 overflow-y-auto transition-all duration-300 ease-in-out"
        :class="[
          isMobile
            ? (sidebarOpen ? 'max-h-[45vh] border-t' : 'max-h-0 border-t-0 p-0 overflow-hidden')
            : (sidebarOpen ? 'w-[280px] lg:w-[320px] border-l' : 'w-0 border-l-0 p-0 overflow-hidden')
        ]"
      >
        <MeshControls />
        <ElementInfo />
      </div>
    </div>

    <!-- Bottom status bar -->
    <footer class="bg-slate-900 border-t border-slate-800 px-3 md:px-6 py-2 flex flex-wrap items-center gap-2 md:gap-6 text-[10px] md:text-xs text-slate-400">
      <span class="flex items-center gap-1">
        应力:
        <span class="text-red-400 font-bold">
          {{ store.result ? (store.maxStress / 1e6).toFixed(2) + ' MPa' : '—' }}
        </span>
      </span>
      <span class="flex items-center gap-1">
        位移:
        <span class="text-amber-400 font-bold">
          {{ store.result ? (store.maxDisplacement * 1000).toFixed(2) + ' mm' : '—' }}
        </span>
      </span>
      <span class="hidden sm:inline">
        节点: <span class="text-slate-200">{{ store.model.nodes.length }}</span>
      </span>
      <span class="hidden sm:inline">
        单元: <span class="text-slate-200">{{ store.model.elements.length }}</span>
      </span>
      <span class="ml-auto text-slate-600 hidden md:inline">
        热力图: {{ store.heatmapMode }}
      </span>
    </footer>
  </div>
</template>
