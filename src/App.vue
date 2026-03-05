<script setup>
import { ref, computed } from 'vue';
import PieChart from './components/Piechart.vue';
import AddFoodDialog from './components/AddFoodDialog.vue';
import ManageUserAddedFoods from './components/ManageUserAddedFoods.vue';
import UserAddRecipeItems from './components/UserAddRecipeItems.vue';
import FoodSummary from './components/FoodSummary.vue';
import NutrionalSummary from './components/NutrionalSummary.vue';
import Menu from './components/Menu.vue';
import TrackingHistory from './components/TrackingHistory.vue';
import foodsData from './data/foods.json';

const FOODS_STORAGE_KEY = 'calcTrackerFoods';
const CALORIE_GOAL_STORAGE_KEY = 'calcTrackerCalorieGoal';
const TRACKED_FOODS_STORAGE_KEY = 'calcTrackerTrackedFoods';
const TRACKING_HISTORY_STORAGE_KEY = 'calcTrackerTrackingHistory';
const DEFAULT_CALORIE_GOAL = 2000;

const trackedFoods = ref(loadTrackedFoods());
const userFoods = ref(loadUserFoods());
const calorieGoal = ref(loadCalorieGoal());
const foods = computed(() => {
  return [...foodsData.foods, ...userFoods.value];
});
const showFoodDialog = ref(false);
const showUserFoodItems = ref(false);
const showRecipeDialog = ref(false);
const showSaveDialog = ref(false);
const showHistory = ref(false);
const saveDate = ref(getTodayDateString());
const trackingHistory = ref(loadTrackingHistory());
const todayDate = new Date().toLocaleDateString(undefined, {
  weekday: 'long',
  year: 'numeric',
  month: 'long',
  day: 'numeric'
});

function getTodayDateString() {
  const today = new Date();
  const year = today.getFullYear();
  const month = String(today.getMonth() + 1).padStart(2, '0');
  const day = String(today.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
}

function loadUserFoods() {
  return loadStoredValue(
    FOODS_STORAGE_KEY,
    [],
    value => Array.isArray(value)
  );
}

function saveUserFoods(updatedUserFoods) {
  saveStoredValue(FOODS_STORAGE_KEY, updatedUserFoods);
}

function loadCalorieGoal() {
  return loadStoredValue(
    CALORIE_GOAL_STORAGE_KEY,
    DEFAULT_CALORIE_GOAL,
    value => Number.isInteger(value) && value > 0
  );
}

function saveCalorieGoal(goal) {
  saveStoredValue(CALORIE_GOAL_STORAGE_KEY, goal);
}

function loadTrackedFoods() {
  const storedFoods = loadStoredValue(
    TRACKED_FOODS_STORAGE_KEY,
    [],
    value => Array.isArray(value)
  );

  return storedFoods.map(item => {
    const parsedUnits = parseInt(item.units ?? item.grams, 10);
    return {
      ...item,
      units: Math.max(1, Number.isNaN(parsedUnits) ? 1 : parsedUnits),
      measurement: item.measurement ?? item.unit
    };
  });
}

function saveTrackedFoods(foods) {
  saveStoredValue(TRACKED_FOODS_STORAGE_KEY, foods);
}

function loadTrackingHistory() {
  return loadStoredValue(
    TRACKING_HISTORY_STORAGE_KEY,
    {},
    value => typeof value === 'object' && value !== null
  );
}

function saveTrackingHistory(history) {
  saveStoredValue(TRACKING_HISTORY_STORAGE_KEY, history);
}

function loadStoredValue(key, fallbackValue, isValid) {
  try {
    const stored = localStorage.getItem(key);
    if (!stored) {
      return fallbackValue;
    }

    const parsed = JSON.parse(stored);
    if (isValid(parsed)) {
      return parsed;
    }
  } catch {
    // Ignore invalid stored data
  }

  return fallbackValue;
}

function saveStoredValue(key, value) {
  try {
    localStorage.setItem(key, JSON.stringify(value));
  } catch {
    // Ignore storage errors
  }
}

// Calculate meal calories for pie chart display
const mealCalories = computed(() => {
  const meals = {
    breakfast: 0,
    lunch: 0,
    dinner: 0,
    snack: 0
  };
  
  trackedFoods.value.forEach(item => {
    const food = foods.value.find(f => f.id === item.foodId);
    if (food && Object.hasOwn(meals, item.mealType)) {
      const parsedUnits = parseInt(item.units ?? item.grams, 10);
      const amount = Math.max(1, Number.isNaN(parsedUnits) ? 1 : parsedUnits);
      const measurement = item.measurement ?? food.unit;
      const multiplier = measurement === 'portion' ? amount : amount / 100;
      meals[item.mealType] += food.calories * multiplier;
    }
  });
  
  return meals;
});

const totalCalories = computed(() => {
  return Object.values(mealCalories.value).reduce((sum, cal) => sum + cal, 0);
});

function handleUpdateFood() {
  showFoodDialog.value = true;
}

function handleTrackedFoodsUpdate(updatedFoods) {
  trackedFoods.value = updatedFoods;
  saveTrackedFoods(updatedFoods);
}

function handleUserFoodsUpdate(updatedFoods) {
  const userOnlyFoods = updatedFoods.filter(f => !foodsData.foods.some(base => base.id === f.id));
  userFoods.value = userOnlyFoods;
  saveUserFoods(userOnlyFoods);
}

function handleCalorieGoalUpdate(newGoal) {
  const goal = parseInt(newGoal, 10);
  if (!Number.isNaN(goal) && goal > 0) {
    calorieGoal.value = goal;
    saveCalorieGoal(goal);
  }
}

function clearDailyRegistrations() {
  trackedFoods.value = [];
  saveTrackedFoods([]);
}

function handleSaveTracking() {
  showSaveDialog.value = true;
}

function handleShowHistory() {
  showHistory.value = true;
}

function confirmSaveTracking() {
  if (!saveDate.value) {
    alert('Please select a date');
    return;
  }

  const history = { ...trackingHistory.value };
  history[saveDate.value] = {
    totalCalories: totalCalories.value,
    trackedFoods: trackedFoods.value
  };
  
  trackingHistory.value = history;
  saveTrackingHistory(history);
  showSaveDialog.value = false;
  alert(`Tracking saved for ${saveDate.value}`);
}

function cancelSaveTracking() {
  showSaveDialog.value = false;
  saveDate.value = getTodayDateString();
}


</script>

<template>
  <main>
    <Menu 
      :trackedFoods="trackedFoods" 
      @manage-foods="showUserFoodItems = true"
      @clear-tracked-foods="clearDailyRegistrations"
      @save-tracking="handleSaveTracking"
      @show-history="handleShowHistory"
    />
    <header>
      <h1 class="title"> Calorie Tracker </h1>
      <p class="today-date">{{ todayDate }}</p>
    </header>
    <section class="dashboard-layout" aria-label="Daily nutrition dashboard">
      <section class="summary-panel" aria-label="Nutritional breakdown">
        <NutrionalSummary :trackedFoods="trackedFoods" :foods="foods" />
      </section>
      <section class="chart-panel" aria-label="Calorie visualization">
        <PieChart :mealCalories="mealCalories" :calorieGoal="calorieGoal" @update:food="handleUpdateFood" @update:calorieGoal="handleCalorieGoalUpdate"/>
      </section>
      <section class="summary-panel" aria-label="Food intake summary">
        <FoodSummary :trackedFoods="trackedFoods" :foods="foods" />
      </section>
    </section>

    <!-- Add Food Dialog -->
    <AddFoodDialog 
      v-if="showFoodDialog"
      :trackedFoods="trackedFoods"
      :foods="foods"
      @update:trackedFoods="handleTrackedFoodsUpdate"
      @update:foods="handleUserFoodsUpdate"
      @open-recipe-dialog="showRecipeDialog = true"
      @close="showFoodDialog = false"
    />

    <!-- User Added Foods Panel -->
    <ManageUserAddedFoods 
      v-if="showUserFoodItems"
      :foods="userFoods"
      :trackedFoods="trackedFoods"
      @update:foods="handleUserFoodsUpdate"
      @update:trackedFoods="handleTrackedFoodsUpdate"
      @close="showUserFoodItems = false"
    />

    <!-- Build Recipe Panel -->
    <UserAddRecipeItems 
      v-if="showRecipeDialog"
      :foods="foods"
      @update:foods="handleUserFoodsUpdate"
      @close="showRecipeDialog = false"
    />

    <!-- Save Tracking Dialog -->
    <div v-if="showSaveDialog" class="dialog-overlay-save" role="dialog" aria-modal="true" aria-labelledby="save-dialog-title">
      <div class="save-dialog">
        <header class="dialog-header">
          <h2 id="save-dialog-title">Save Current Tracking</h2>
          <button class="close-btn" @click="cancelSaveTracking" aria-label="Close dialog">&times;</button>
        </header>
        <div class="dialog-content">
          <p class="total-calories">Total Calories: <strong>{{ Math.round(totalCalories) }}</strong></p>
          <div class="form-group">
            <label for="save-date-input">Select Date:</label>
            <input 
              id="save-date-input"
              v-model="saveDate" 
              type="date" 
              class="date-input"
            />
          </div>
          <p class="info-text">Note: Saving to an existing date will overwrite the previous data.</p>
        </div>
        <div class="dialog-buttons">
          <button class="save-btn" @click="confirmSaveTracking">Save</button>
          <button class="cancel-btn" @click="cancelSaveTracking">Cancel</button>
        </div>
      </div>
    </div>

    <!-- Tracking History -->
    <TrackingHistory
      v-if="showHistory"
      :trackingHistory="trackingHistory"
      :foods="foods"
      @close="showHistory = false"
    />

    <footer class="app-footer">
      <p>
        Food data reference: Marija Langwagen, Jette Jacobsen, Tue Christensen and Anders Poulsen: The Danish Food Composition Database, Frida version 5.5, November 2025, National Food Institute, Technical University of Denmark.
        <a href="https://doi.org/10.11583/DTU.29500682" target="_blank" rel="noopener noreferrer">https://doi.org/10.11583/DTU.29500682</a>
      </p>
    </footer>
  </main>
</template>

<style scoped>

main {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}


.title{
  width: 100vw;
  text-align: center;
  padding-top: 10px;
  font-size: xx-large;
}

.today-date {
  width: 100vw;
  text-align: center;
  margin-top: 4px;
  margin-bottom: 8px;
  color: var(--text-color);
  opacity: 0.85;
  font-size: 14px;
}

.dashboard-layout {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 20px;
  padding: 20px 16px;
  width: 100%;
  max-width: 1280px;
  margin: 0 auto;
  flex: 1;
}

.chart-panel {
  display: flex;
  justify-content: center;
  align-items: center;
  order: -1;
  width: 100%;
  max-width: 100%;
}

.summary-panel {
  width: 100%;
  max-width: 100%;
}

@media (min-width: 1000px) {
  .dashboard-layout {
    flex-direction: row;
    align-items: center;
    justify-content: center;
    gap: 20px;
    padding: 20px 16px;
  }

  .chart-panel {
    flex: 1 1 0;
    align-items: center;
    order: 0;
    max-width: 33.333vw;
  }

  .summary-panel {
    flex: 1 1 0;
    max-width: 33.333vw;
  }
}

.app-footer {
  width: 100%;
  margin: 12px auto 20px;
  text-align: center;
  padding: 12px 16px;
  border-top: 1px solid var(--border-color);
  color: var(--text-muted-color);
  font-size: 12px;
  line-height: 1.5;
}

.app-footer a {
  color: var(--primary-color);
  text-decoration: underline;
}

.app-footer a:hover {
  color: var(--primary-hover-color);
}

.dialog-overlay-save {
  position: fixed;
  top:0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: var(--overlay-color);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.save-dialog {
  background-color: var(--bg-color);
  border-radius: 12px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 4px 20px var(--shadow-color);
}

.dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px;
  border-bottom: 1px solid var(--border-color);
}

.dialog-header h2 {
  margin: 0;
  font-size: 20px;
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

.dialog-content {
  padding: 24px;
}

.total-calories {
  font-size: 18px;
  margin-bottom: 20px;
  color: var(--text-color);
}

.total-calories strong {
  color: var(--primary-color);
  font-size: 24px;
}

.form-group {
  margin-bottom: 16px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  color: var(--text-color);
  font-weight: 500;
}

.date-input {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid var(--border-color);
  border-radius: 6px;
  background-color: var(--surface-color);
  color: var(--text-color);
  font-size: 16px;
}

.date-input:focus {
  outline: none;
  border-color: var(--primary-color);
}

.info-text {
  color: var(--text-muted-color);
  font-size: 14px;
  margin-top: 12px;
  font-style: italic;
}

.dialog-buttons {
  display: flex;
  gap: 12px;
  padding: 20px 24px;
  border-top: 1px solid var(--border-color);
  justify-content: flex-end;
}

.save-btn,
.cancel-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  transition: background-color 0.2s;
}

.save-btn {
  background-color: var(--primary-color);
  color: white;
}

.save-btn:hover {
  background-color: var(--primary-hover-color);
}

.cancel-btn {
  background-color: var(--surface-alt-color);
  color: var(--text-color);
}

.cancel-btn:hover {
  background-color: var(--border-color);
}

</style>
