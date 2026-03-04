<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  mealCalories: {
    type: Object,
    default: () => ({ breakfast: 0, lunch: 0, dinner: 0, snack: 0 })
  },
  calorieGoal: {
    type: Number,
    default: 2000
  }
});

const emits = defineEmits(['update:food', 'update:calorieGoal']);

const showGoalEditDialog = ref(false);
const tempGoalValue = ref(props.calorieGoal);

defineExpose({
  getCalorieStats
});

// Calculate total calories from meal calories object
const current_calories = computed(() => {
  return props.mealCalories.breakfast + props.mealCalories.lunch + props.mealCalories.dinner + props.mealCalories.snack;
});

// Calculate pie chart gradient stops
const pieChartGradient = computed(() => {
  const total = current_calories.value;
  
  // If no calories tracked, show white
  if (total === 0) {
    return 'var(--surface-alt-color) 0 100%';
  }
  
  // Calculate percentages for each meal category
  const breakfastPct = (props.mealCalories.breakfast / total) * 100;
  const lunchPct = (props.mealCalories.lunch / total) * 100;
  const dinnerPct = (props.mealCalories.dinner / total) * 100;
  const snackPct = (props.mealCalories.snack / total) * 100;
  
  // Build gradient stops
  const stops = [];
  let currentStop = 0;
  
  if (props.mealCalories.breakfast > 0) {
    stops.push(`var(--meal-breakfast-color) ${currentStop}% ${currentStop + breakfastPct}%`);
    currentStop += breakfastPct;
  }
  
  if (props.mealCalories.lunch > 0) {
    stops.push(`var(--meal-lunch-color) ${currentStop}% ${currentStop + lunchPct}%`);
    currentStop += lunchPct;
  }
  
  if (props.mealCalories.dinner > 0) {
    stops.push(`var(--meal-dinner-color) ${currentStop}% ${currentStop + dinnerPct}%`);
    currentStop += dinnerPct;
  }
  
  if (props.mealCalories.snack > 0) {
    stops.push(`var(--meal-snack-color) ${currentStop}% ${currentStop + snackPct}%`);
  }
  
  return stops.join(', ');
});

function getCalorieStats() {
  return {
    calories: Math.round(current_calories.value * 100) / 100,
    meals: props.mealCalories
  };
}

function openGoalEditDialog() {
  tempGoalValue.value = props.calorieGoal;
  showGoalEditDialog.value = true;
}

function saveCalorieGoal() {
  emits('update:calorieGoal', tempGoalValue.value);
  showGoalEditDialog.value = false;
}

function cancelGoalEdit() {
  showGoalEditDialog.value = false;
}
</script>

<template>
    <div class="progress-wrapper" 
        :style="{ '--gradient': pieChartGradient }"
        role="img"
        :aria-label="`Calorie breakdown: ${Math.round(current_calories)} calories total. Breakfast: ${Math.round(mealCalories.breakfast)}, Lunch: ${Math.round(mealCalories.lunch)}, Dinner: ${Math.round(mealCalories.dinner)}, Snack: ${Math.round(mealCalories.snack)}. Total: ${Math.round(current_calories)}/${calorieGoal} calories`"
      >
        <div class="progress-text-container">
          <div class="progress-text" @click="openGoalEditDialog" title="Click to edit calorie goal">{{Math.round(current_calories)}}/{{calorieGoal}}</div>
        </div>
        <button :style="'z-index: 1;'" @click="emits('update:food')">Update foods</button>
    </div>

    <!-- Calorie Goal Edit Dialog -->
    <div v-if="showGoalEditDialog" class="goal-edit-overlay">
      <div class="goal-edit-dialog">
        <h3>Set Calorie Goal</h3>
        <input 
          v-model.number="tempGoalValue" 
          type="number" 
          min="100" 
          max="10000"
          class="goal-input"
          placeholder="Enter daily calorie goal"
        />
        <div class="dialog-buttons">
          <button class="btn-save" @click="saveCalorieGoal">Save</button>
          <button class="btn-cancel" @click="cancelGoalEdit">Cancel</button>
        </div>
      </div>
    </div>
</template>

<style scoped>  


.progress-wrapper {
    position: relative;
    width: 100%;
    aspect-ratio: 1 / 1;
    border-radius: 50%;
    /* Dynamic pie chart colors based on meal categories */
    background: conic-gradient(var(--gradient));
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
}

@media (min-width: 650px) {
  .progress-wrapper {
    max-width: 33.333vw
  }
}

/* Creates the inner circle to form a ring */
.progress-wrapper::before {
    content: '';
    position: absolute;
    width: 80%;
    height: 80%;
    border-radius: 50%;
  background-color: var(--surface-color);
}

.progress-text {
    position: relative;
    z-index: 1; /* Keeps text above the colored gradient */
    font-size: 30px;
    color: var(--text-color);
    cursor: pointer;
    padding: 8px 12px;
    border-radius: 4px;
    transition: background-color 0.2s ease;
}

.progress-text:hover {
    background-color: rgba(0, 0, 0, 0.1);
}

.progress-text-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
}

.goal-edit-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
  background-color: var(--overlay-color);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 2000;
}

.goal-edit-dialog {
    background-color: var(--surface-color);
    border: 1px solid var(--border-color);
    border-radius: 8px;
    padding: 24px;
    min-width: 300px;
  box-shadow: 0 4px 12px var(--shadow-color);
}

.goal-edit-dialog h3 {
    margin: 0 0 16px;
    color: var(--text-color);
    font-size: 18px;
}

.goal-input {
    width: 100%;
    padding: 10px 12px;
    border: 1px solid var(--border-color);
    border-radius: 4px;
    background-color: var(--input-bg);
    color: var(--text-color);
    font-size: 16px;
    margin-bottom: 20px;
    box-sizing: border-box;
}

.goal-input:focus {
    outline: none;
    border-color: var(--primary-color);
    box-shadow: 0 0 0 2px var(--focus-ring-color);
}

.btn-save, .btn-cancel {
    padding: 8px 16px;
    border: none;
    border-radius: 4px;
    font-size: 14px;
    cursor: pointer;
    transition: background-color 0.2s ease;
}

.btn-save {
    background-color: var(--primary-color);
    color: white;
}

.btn-save:hover {
    background-color: var(--primary-hover-color);
}

.btn-cancel {
    background-color: var(--surface-alt-color);
    color: var(--text-color);
}

.btn-cancel:hover {
    background-color: var(--border-color);
}

</style>