<script setup>
import { ref, computed } from 'vue';
import NutrionalSummary from './NutrionalSummary.vue';

const props = defineProps({
  trackingHistory: {
    type: Object,
    required: true
  },
  foods: {
    type: Array,
    required: true
  }
});

const emit = defineEmits(['close']);

const selectedDate = ref(null);

const historyEntries = computed(() => {
  return Object.entries(props.trackingHistory)
    .map(([date, data]) => ({
      date,
      calories: data.totalCalories,
      trackedFoods: data.trackedFoods
    }))
    .sort((a, b) => new Date(a.date) - new Date(b.date));
});

const maxCalories = computed(() => {
  if (historyEntries.value.length === 0) return 2500;
  return Math.max(...historyEntries.value.map(e => e.calories), 2500);
});

const selectedEntry = computed(() => {
  if (!selectedDate.value) return null;
  return historyEntries.value.find(e => e.date === selectedDate.value);
});

function selectDate(date) {
  selectedDate.value = date;
}

function clearSelection() {
  selectedDate.value = null;
}

function formatDate(dateString) {
  const date = new Date(dateString);
  return date.toLocaleDateString(undefined, { month: 'short', day: 'numeric', year: 'numeric' });
}

function formatShortDate(dateString) {
  const date = new Date(dateString);
  return date.toLocaleDateString(undefined, { month: 'short', day: 'numeric' });
}

function getBarHeight(calories) {
  return Math.min((calories / maxCalories) * 100, 100);
}

function handleClose() {
  emit('close');
}
</script>

<template>
  <div class="history-overlay" role="dialog" aria-modal="true" aria-labelledby="history-title">
    <section class="history-container">
      <header class="history-header">
        <h2 id="history-title">Tracking History</h2>
        <button class="close-btn" @click="handleClose" aria-label="Close history">&times;</button>
      </header>

      <div v-if="historyEntries.length === 0" class="no-history" role="status">
        No tracking history available. Save your current tracking to start building history.
      </div>

      <div v-else class="history-content">
        <!-- Timeline Chart -->
        <section class="chart-section" aria-label="Calorie history chart">
          <h3 class="visually-hidden">Daily Calorie Chart</h3>
          <div class="chart-container">
            <div class="chart-y-axis" aria-hidden="true">
              <span>{{ Math.round(maxCalories) }}</span>
              <span>{{ Math.round(maxCalories * 0.75) }}</span>
              <span>{{ Math.round(maxCalories * 0.5) }}</span>
              <span>{{ Math.round(maxCalories * 0.25) }}</span>
              <span>0</span>
            </div>
            <div class="chart-bars-container">
              <div class="chart-bars">
                <button
                  v-for="entry in historyEntries"
                  :key="entry.date"
                  class="bar-wrapper"
                  :class="{ selected: selectedDate === entry.date }"
                  @click="selectDate(entry.date)"
                  :aria-label="`${formatDate(entry.date)}: ${Math.round(entry.calories)} calories`"
                >
                  <div class="bar" :style="{ height: getBarHeight(entry.calories) + '%' }">
                    <span class="bar-value">{{ Math.round(entry.calories) }}</span>
                  </div>
                  <span class="bar-label">{{ formatShortDate(entry.date) }}</span>
                </button>
              </div>
            </div>
          </div>
        </section>

        <!-- Nutritional Details -->
        <section v-if="selectedEntry" class="details-section" aria-labelledby="details-heading">
          <div class="details-header">
            <h3 id="details-heading">{{ formatDate(selectedEntry.date) }}</h3>
            <button class="clear-selection-btn" @click="clearSelection" aria-label="Clear date selection">Clear Selection</button>
          </div>
          <NutrionalSummary :trackedFoods="selectedEntry.trackedFoods" :foods="foods" />
        </section>

        <div v-else class="details-placeholder" role="status">
          Click on a date to view nutritional details
        </div>
      </div>
    </section>
</div>
</template>

<style scoped>
.history-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: var(--overlay-color);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.history-container {
  background-color: var(--bg-color);
  border-radius: 12px;
  width: 100%;
  max-width: 1280px;
  height: 100vh;
  display: flex;
  flex-direction: column;
  box-shadow: 0 4px 20px var(--shadow-color);
}

.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px;
  border-bottom: 1px solid var(--border-color);
}

.history-header h2 {
  margin: 0;
  font-size: 24px;
  color: var(--text-color);
}

.close-btn {
  background: transparent;
  border: none;
  font-size: 32px;
  color: var(--text-color);
  cursor: pointer;
  padding: 0;
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
}

.close-btn:hover {
  background-color: var(--surface-alt-color);
}

.history-content {
  padding: 24px;
  overflow-y: auto;
  flex: 1;
}

.no-history {
  padding: 40px;
  text-align: center;
  color: var(--text-muted-color);
}

.chart-section {
  margin-bottom: 32px;
}

.chart-container {
  display: flex;
  gap: 12px;
  align-items: stretch;
}

.chart-y-axis {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 10px 8px;
  color: var(--text-muted-color);
  font-size: 12px;
  text-align: right;
  min-width: 50px;
}

.chart-bars-container {
  flex: 1;
  overflow: auto;
  overflow-y: hidden;
  padding: 10px 0;
}

.chart-bars {
  display: flex;
  gap: 12px;
  align-items: flex-end;
  min-height: 250px;
  padding: 0 12px;
}

.bar-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  min-width: 60px;
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 0;
  transition: transform 0.2s;
}

.bar-wrapper:hover {
  transform: translateY(-10px);
}

.bar-wrapper.selected .bar {
  background-color: var(--primary-color);
}

.bar {
  width: 100%;
  min-height: 20px;
  background-color: var(--surface-alt-color);
  border-radius: 4px 4px 0 0;
  position: relative;
  transition: background-color 0.2s;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding-top: 4px;
}

.bar:hover {
  background-color: var(--primary-hover-color);
}

.bar-value {
  font-size: 11px;
  color: var(--text-color);
  font-weight: 600;
}

.bar-label {
  font-size: 11px;
  color: var(--text-muted-color);
  text-align: center;
  word-wrap: break-word;
  max-width: 60px;
}

.details-section {
  background-color: var(--surface-color);
  border-radius: 8px;
  padding: 20px;
  border: 1px solid var(--border-color);
}

.details-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.details-header h3 {
  margin: 0;
  font-size: 20px;
  color: var(--text-color);
}

.clear-selection-btn {
  padding: 6px 12px;
  background-color: var(--surface-alt-color);
  border: 1px solid var(--border-color);
  border-radius: 4px;
  color: var(--text-color);
  cursor: pointer;
  font-size: 14px;
}

.clear-selection-btn:hover {
  background-color: var(--primary-hover-color);
}

.details-placeholder {
  text-align: center;
  padding: 40px;
  color: var(--text-muted-color);
  font-size: 16px;
  font-style: italic;
}

@media (max-width: 768px) {
  .history-overlay {
    padding: 0;
  }

  .history-container {
    border-radius: 0;
    max-height: 100vh;
    height: 100vh;
  }

  .chart-bars {
    min-height: 200px;
  }

  .bar-wrapper {
    min-width: 50px;
  }
}
</style>
