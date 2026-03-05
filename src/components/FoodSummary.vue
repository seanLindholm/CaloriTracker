<script setup>
import { computed } from 'vue';

const props = defineProps({
  trackedFoods: {
    type: Array,
    required: true
  },
  foods: {
    type: Array,
    required: true
  }
});

const mealOrder = ['breakfast', 'lunch', 'dinner', 'snack'];

const summaryByMeal = computed(() => {
  const meals = {
    breakfast: [],
    lunch: [],
    dinner: [],
    snack: []
  };

  props.trackedFoods.forEach(item => {
    const food = props.foods.find(f => f.id === item.foodId);
    const measurement = item.measurement ?? (food ? food.unit : 'measurement');
    const name = food ? food.name : item.foodName || item.foodId;
    const parsedUnits = parseInt(item.units ?? item.grams, 10);
    const amount = Math.max(1, Number.isNaN(parsedUnits) ? 1 : parsedUnits);
    const calories = food
      ? measurement === 'portion'
        ? food.calories * amount
        : (food.calories * amount) / 100
      : 0;

    if (meals[item.mealType]) {
      meals[item.mealType].push({
        id: item.foodId,
        name,
        units: amount,
        measurement,
        calories
      });
    }
  });

  return meals;
});

const hasAnyFoods = computed(() => {
  return props.trackedFoods.length > 0;
});
</script>

<template>
  <div class="food-summary">
    <h2>Food Summary</h2>

    <div v-if="!hasAnyFoods" class="no-foods">
      No foods added yet
    </div>

    <div v-else class="food-summary-scroll themed-scrollbar">
      <table class="food-summary-table">
        <thead>
          <tr>
            <th>Food</th>
            <th>Amount</th>
            <th>Calories</th>
          </tr>
        </thead>
        <tbody>
          <template v-for="meal in mealOrder" :key="meal">
            <tr v-if="summaryByMeal[meal].length" class="meal-banner">
              <td colspan="3">{{ meal.charAt(0).toUpperCase() + meal.slice(1) }}</td>
            </tr>
            <tr v-for="item in summaryByMeal[meal]" :key="meal + '-' + item.id">
              <td>{{ item.name }}</td>
              <td>{{ item.units }} {{ item.measurement }}</td>
              <td>{{ Math.round(item.calories * 100) / 100 }}</td>
            </tr>
          </template>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.food-summary {
  padding: 10px;
  max-width: 1280px;
  margin-left: auto;
  margin-right: auto;
}

.food-summary h2 {
  font-size: 20px;
  color: var(--text-color);
}

.meal-banner td {
  background-color: var(--surface-alt-color);
  font-weight: 600;
  text-transform: capitalize;
}

@media (min-width: 1000px) {
  .food-summary-scroll {
    max-height: var(--summary-scroll-max-height);
    overflow-y: auto;
    scrollbar-gutter: stable;
    border-radius: 6px;
  }
}
</style>
