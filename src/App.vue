<script setup>
import { ref, computed } from 'vue';
import PieChart from './components/Piechart.vue';
import AddFoodDialog from './components/AddFoodDialog.vue';
import ManageUserAddedFoods from './components/ManageUserAddedFoods.vue';
import UserAddRecipeItems from './components/UserAddRecipeItems.vue';
import FoodSummary from './components/FoodSummary.vue';
import NutrionalSummary from './components/NutrionalSummary.vue';
import foodsData from './data/foods.json';

const FOODS_STORAGE_KEY = 'calcTrackerFoods';
const CALORIE_GOAL_STORAGE_KEY = 'calcTrackerCalorieGoal';
const TRACKED_FOODS_STORAGE_KEY = 'calcTrackerTrackedFoods';
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
const todayDate = new Date().toLocaleDateString(undefined, {
  weekday: 'long',
  year: 'numeric',
  month: 'long',
  day: 'numeric'
});

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


</script>

<template>
  <main>
    <h1 class="title"> Calorie Tracker </h1>
    <p class="today-date">{{ todayDate }}</p>
    <button class="clear-daily-btn" @click="clearDailyRegistrations" title="Clear today's registrations">
      <svg class="trash-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M3 6h18"></path>
        <path d="M8 6V4h8v2"></path>
        <path d="M19 6l-1 14H6L5 6"></path>
        <path d="M10 11v6"></path>
        <path d="M14 11v6"></path>
      </svg>
    </button>
    <button class="manage-foods-btn" @click="showUserFoodItems = !showUserFoodItems" title="Manage foods">
      <svg class="pencil-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path>
        <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path>
      </svg>
    </button>
    <div class="dashboard-layout" alt="main dashboard layout">
      <div class="summary-panel">
        <NutrionalSummary :trackedFoods="trackedFoods" :foods="foods" />
      </div>
      <div class="chart-panel">
        <PieChart :mealCalories="mealCalories" :calorieGoal="calorieGoal" @update:food="handleUpdateFood" @update:calorieGoal="handleCalorieGoalUpdate"/>
      </div>
      <div class="summary-panel">
        <FoodSummary :trackedFoods="trackedFoods" :foods="foods" />
      </div>
    </div>

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


.manage-foods-btn,
.clear-daily-btn {
  padding: 10px;
  border: none;
  border-radius: 40px;  
  cursor: pointer;
  position: fixed;
  top: 10px;
  z-index: 100;
  display: flex;
  align-items: center;
  justify-content: center;
}

.manage-foods-btn {
  right: max(10px, calc((100vw - 1280px) / 2));
}

.clear-daily-btn {
  left: max(10px, calc((100vw - 1280px) / 2));
}

.clear-daily-btn:hover {
  background-color: var(--danger-color);
}

.manage-foods-btn:hover {
  background-color: var(--primary-hover-color);
}

.pencil-icon {
  width: 20px;
  height: 20px;
}

.trash-icon {
  width: 20px;
  height: 20px;
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

</style>
