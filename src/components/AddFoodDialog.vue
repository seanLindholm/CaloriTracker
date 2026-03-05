<script setup>
import { ref, computed } from 'vue';

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

const emits = defineEmits(['update:trackedFoods', 'update:foods', 'close', 'open-recipe-dialog']);

const selectedMealType = ref('breakfast');
const showAddDialog = ref(false);
const foodSearch = ref('');
const selectedFoodId = ref('');
const isSearchFocused = ref(false);
const foodAmount = ref(null);
const isClosing = ref(false);

const showAddNewDialog = ref(false);
const newFoodName = ref('');
const newFoodUnit = ref('gram');
const newFoodCalories = ref('');
const newFoodProtein = ref('');
const newFoodCarbohydrates = ref('');
const newFoodFat = ref('');
const newFoodSalt = ref('');

const mealTypes = ['breakfast', 'lunch', 'dinner', 'snack'];

const mealFoods = computed(() => {
  return props.trackedFoods.filter(item => item.mealType === selectedMealType.value);
});

const filteredFoods = computed(() => {
  if (!foodSearch.value.trim()) {
    return props.foods;
  }
  const searchLower = foodSearch.value.toLowerCase();
  return props.foods.filter(food =>
    food.name.toLowerCase().includes(searchLower)
  );
});

const selectedFood = computed(() => {
  if (selectedFoodId.value) {
    return props.foods.find(food => food.id === selectedFoodId.value) || null;
  }
  return props.foods.find(food => food.name === foodSearch.value) || null;
});

const selectedFoodUnit = computed(() => {
  return selectedFood.value ? selectedFood.value.unit : 'unit';
});

function updateFoodAmount(index, newAmount) {
  const mealFoodsIndices = props.trackedFoods
    .map((item, idx) => item.mealType === selectedMealType.value ? idx : -1)
    .filter(idx => idx !== -1);

  const globalIndex = mealFoodsIndices[index];
  const updatedFoods = [...props.trackedFoods];
  updatedFoods[globalIndex].units = Math.max(1, parseInt(newAmount) || 0);
  emits('update:trackedFoods', updatedFoods);
}

function removeFoodItem(index) {
  const mealFoodsIndices = props.trackedFoods
    .map((item, idx) => item.mealType === selectedMealType.value ? idx : -1)
    .filter(idx => idx !== -1);

  const globalIndex = mealFoodsIndices[index];
  const updatedFoods = props.trackedFoods.filter((_, idx) => idx !== globalIndex);
  emits('update:trackedFoods', updatedFoods);
}

function addFoodItem() {
  if (!selectedFood.value) {
    return;
  }

  const amount = Math.max(1, parseInt(foodAmount.value) || 100);

  const existingIndex = props.trackedFoods.findIndex(
    item => item.foodId === selectedFood.value.id && item.mealType === selectedMealType.value
  );

  const updatedFoods = [...props.trackedFoods];

  if (existingIndex >= 0) {
    const parsedUnits = parseInt(updatedFoods[existingIndex].units ?? updatedFoods[existingIndex].grams, 10);
    const currentUnits = Math.max(1, Number.isNaN(parsedUnits) ? 1 : parsedUnits);
    updatedFoods[existingIndex].units = currentUnits + amount;
    updatedFoods[existingIndex].measurement = selectedFood.value.unit;
  } else {
    updatedFoods.push({
      foodId: selectedFood.value.id,
      foodName: selectedFood.value.name,
      mealType: selectedMealType.value,
      units: amount,
      measurement: selectedFood.value.unit
    });
  }

  emits('update:trackedFoods', updatedFoods);

  foodSearch.value = '';
  selectedFoodId.value = '';
  foodAmount.value = null;
  showAddDialog.value = false;
}

function selectFood(food) {
  selectedFoodId.value = food.id;
  foodSearch.value = food.name;
  isSearchFocused.value = false;
}

function handleSearchFocus() {
  isSearchFocused.value = true;
}

function handleSearchBlur() {
  setTimeout(() => {
    isSearchFocused.value = false;
  }, 100);
}

function getNextAvailableFoodId() {
  const numericIds = props.foods
    .map(food => Number(food.id))
    .filter(id => Number.isInteger(id) && id > 0);

  if (numericIds.length === 0) {
    return 1;
  }

  return Math.max(...numericIds) + 1;
}

function addNewFood() {
  const name = newFoodName.value.trim();
  if (!name) {
    return;
  }

  const id = getNextAvailableFoodId();
  const calories = parseFloat(newFoodCalories.value);
  const protein = parseFloat(newFoodProtein.value);
  const carbohydrates = parseFloat(newFoodCarbohydrates.value);
  const fat = parseFloat(newFoodFat.value);
  const salt = parseFloat(newFoodSalt.value);

  if (Number.isNaN(calories) || Number.isNaN(protein) || Number.isNaN(carbohydrates) || Number.isNaN(fat) || Number.isNaN(salt)) {
    return;
  }

  const exists = props.foods.some(food => Number(food.id) === id);
  if (exists) {
    return;
  }

  const newFood = {
    id,
    name,
    unit: newFoodUnit.value,
    calories,
    protein,
    carbohydrates,
    fat,
    salt,
    fibre: 0,
    iron: 0,
    vitaminA: 0,
    vitaminB1: 0,
    vitaminB2: 0,
    vitaminB6: 0,
    vitaminB12: 0,
    vitaminC: 0,
    vitaminD: 0,
    vitaminD2: 0,
    vitaminD3: 0,
    vitaminE: 0,
    vitaminK: 0,
    vitaminK1: 0,
    vitaminK2: 0
  };

  const updatedFoods = [...props.foods, newFood];
  emits('update:foods', updatedFoods);

  selectedFoodId.value = id;
  foodSearch.value = name;
  showAddNewDialog.value = false;
  newFoodName.value = '';
  newFoodUnit.value = 'gram';
  newFoodCalories.value = '';
  newFoodProtein.value = '';
  newFoodCarbohydrates.value = '';
  newFoodFat.value = '';
  newFoodSalt.value = '';
}

function handleClose() {
  isClosing.value = true;
  setTimeout(() => {
    emits('close');
  }, 300);
}
</script>

<template>
  <div class="dialog-overlay" :class="{ 'closing': isClosing }" role="dialog" aria-modal="true" aria-labelledby="dialog-title">
    <div class="dialog-content">
      <header class="dialog-header">
        <h2 id="dialog-title">Add / Edit Foods</h2>
        <button class="close-btn" @click="handleClose" aria-label="Close dialog">&times;</button>
      </header>

      <div class="meal-selector">
        <label for="meal-type-select" class="visually-hidden">Select meal type</label>
        <select v-model="selectedMealType" id="meal-type-select" class="meal-dropdown">
          <option v-for="meal in mealTypes" :key="meal" :value="meal">
            {{ meal.charAt(0).toUpperCase() + meal.slice(1) }}
          </option>
        </select>
        <button class="add-btn" @click="showAddDialog = true" aria-label="Add new food item">+</button>
      </div>

      <section class="meal-foods" aria-label="Foods for selected meal">
        <div v-if="mealFoods.length === 0" class="no-foods">
          No foods added for {{ selectedMealType }}
        </div>
        <table v-else class="foods-table">
          <thead>
            <tr>
              <th>Food</th>
              <th>Amount</th>
              <th>Action</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in mealFoods" :key="index">
              <td>{{ item.foodName }}</td>
              <td>
                <div class="amount-with-unit">
                  <input
                    type="number"
                    :value="item.units ?? item.grams ?? 1"
                    @change="event => updateFoodAmount(index, event.target.value)"
                    class="amount-input"
                    min="1"
                    :aria-label="`Amount of ${item.foodName}`"
                  />
                  <span class="unit-display">{{ item.measurement || props.foods.find(f => f.id === item.foodId)?.unit || 'unit' }}</span>
                </div>
              </td>
              <td>
                <button class="remove-btn" @click="removeFoodItem(index)" :aria-label="`Remove ${item.foodName}`">Remove</button>
              </td>
            </tr>
          </tbody>
        </table>
      </section>

      <div v-if="showAddDialog" class="add-dialog-overlay" role="dialog" aria-modal="true" aria-labelledby="add-food-title">
        <div class="add-dialog-box">
          <h3 id="add-food-title">Add Food</h3>

          <form @submit.prevent="addFoodItem">
            <div class="form-group">
            <label for="food-search-input">Search Food:</label>
            <input
              v-model="foodSearch"
              required=""
              type="text"
              placeholder="Type food name..."
              class="search-input"
              @focus="handleSearchFocus"
              @blur="handleSearchBlur"
              @input="selectedFoodId = ''"
            />
            <div class="dropdown-list themed-scrollbar" v-if="isSearchFocused && foodSearch.trim() && filteredFoods.length > 0" role="listbox" aria-label="Food search results">
              <button
                v-for="food in filteredFoods"
                :key="food.id"
                type="button"
                class="dropdown-option"
                role="option"
                @mousedown.prevent="selectFood(food)"
              >
                {{ food.name }}
              </button>
            </div>
            <div class="button-group">
              <button
                type="button"
                class="add-new-btn"
                @click="showAddNewDialog = true"
              >
                Add new food
              </button>
              <button
                type="button"
                class="add-new-btn"
                @click="emits('open-recipe-dialog')"
              >
                Build recipe
              </button>
            </div>
            </div>

            <div class="form-group">
            <label for="food-amount-input">Amount ({{ selectedFoodUnit }}):</label>
            <input
              v-model="foodAmount"
              type="number"
              class="amount-input"
              min="1"
              required
            />
            </div>

            <div class="dialog-buttons">
            <button type="submit" class="ok-btn">OK</button>
            <button type="button" class="cancel-btn" @click="showAddDialog = false">Cancel</button>
            </div>
          </form>
        </div>
      </div>

      <div v-if="showAddNewDialog" class="add-dialog-overlay" role="dialog" aria-modal="true" aria-labelledby="new-food-title">
        <div class="add-dialog-box">
          <h3 id="new-food-title">Add New Food</h3>

          <form @submit.prevent="addNewFood">
            <div class="form-group">
              <label for="new-food-name">Name:</label>
              <input id="new-food-name" v-model="newFoodName" type="text" class="search-input" placeholder="Food name" required />
            </div>

            <div class="form-group">
              <label for="new-food-unit">Unit:</label>
              <select id="new-food-unit" v-model="newFoodUnit" class="meal-dropdown">
                <option value="gram">gram</option>
                <option value="ml">ml</option>
                <option value="portion">portion</option>
              </select>
            </div>

            <div class="form-group">
              <label for="new-food-calories">Calories (per 100 units):</label>
              <input id="new-food-calories" v-model="newFoodCalories" type="number" class="amount-input" min="0" required />
            </div>

            <div class="form-group">
              <label for="new-food-protein">Protein (per 100 units):</label>
              <input id="new-food-protein" v-model="newFoodProtein" type="number" class="amount-input" min="0" required />
            </div>

            <div class="form-group">
              <label for="new-food-carbs">Carbohydrates (per 100 units):</label>
              <input id="new-food-carbs" v-model="newFoodCarbohydrates" type="number" class="amount-input" min="0" required />
            </div>

            <div class="form-group">
              <label for="new-food-fat">Fat (per 100 units):</label>
              <input id="new-food-fat" v-model="newFoodFat" type="number" class="amount-input" min="0" required />
            </div>

            <div class="form-group">
              <label for="new-food-salt">Salt (per 100 units):</label>
              <input id="new-food-salt" v-model="newFoodSalt" type="number" class="amount-input" min="0" required />
            </div>

            <div class="dialog-buttons">
              <button type="submit" class="ok-btn">OK</button>
              <button type="button" class="cancel-btn" @click="showAddNewDialog = false">Cancel</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dialog-overlay {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: 65vh;
  z-index: 2;
  border-top: 1px solid var(--border-color);
  background-color: var(--surface-color);
  animation: slideUp 0.3s ease-out;
}

@keyframes slideUp {
  from {
    transform: translateY(100%);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

@keyframes slideDown {
  from {
    transform: translateY(0);
    opacity: 1;
  }
  to {
    transform: translateY(100%);
    opacity: 0;
  }
}

.dialog-overlay.closing {
  animation: slideDown 0.3s ease-in forwards;
}

.dialog-content {
  background-color: var(--surface-color);
  height: 100%;
  display: flex;
  flex-direction: column;
  padding: 20px;
  overflow-y: auto;
  width: 100%;
  max-width: 1280px;
  margin: 0 auto;
}

.dialog-header {
  margin-bottom: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.meal-selector {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  align-items: center;
}

.meal-dropdown {
  flex: 1;
}

.add-btn {
  padding: 10px 16px;
  background-color: var(--success-color);
  color: white;
  font-size: 18px;
}

.add-btn:hover {
  background-color: var(--success-hover-color);
}

.meal-foods {
  flex: 1;
  overflow-y: auto;
}

.foods-table {
  border: 1px solid var(--border-color);
}

.amount-input {
  width: 80px;
  text-align: center;
}

.amount-with-unit {
  display: flex;
  align-items: center;
  gap: 8px;
}

.unit-display {
  color: var(--text-muted-color);
  white-space: nowrap;
}

.remove-btn {
  padding: 6px 12px;
  background-color: var(--danger-color);
  color: white;
}

.remove-btn:hover {
  background-color: var(--danger-hover-color);
}

.add-dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: var(--overlay-color);
  display: flex;
  align-items: center;
  justify-content: center;
}

.add-dialog-box {
  background-color: var(--surface-color);
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 4px 12px var(--shadow-color);
  max-width: 400px;
  width: 90%;
}

.add-dialog-box h3 {
  margin-top: 0;
  margin-bottom: 20px;
  color: var(--text-color);
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
}

.dropdown-list {
  margin-top: 8px;
  border: 1px solid var(--border-color);
  border-radius: 4px;
  background-color: var(--surface-color);
  max-height: 160px;
  overflow-y: auto;
}

.dropdown-option {
  width: 100%;
  text-align: left;
  padding: 8px 10px;
  background: none;
}

.dropdown-option:hover {
  background-color: var(--table-row-hover);
}

.add-new-btn {
  margin-top: 8px;
  background-color: transparent;
  border: 1px dashed var(--border-color);
}

.add-new-btn:hover {
  background-color: var(--table-row-hover);
}

.button-group {
  display: flex;
  gap: 10px;
  margin-top: 8px;
}

.button-group .add-new-btn {
  flex: 1;
  margin-top: 0;
}

.ok-btn {
  background-color: var(--success-color);
  color: white;
}

.ok-btn:hover {
  background-color: var(--success-hover-color);
}

.cancel-btn {
  background-color: var(--secondary-color);
  color: white;
}

.cancel-btn:hover {
  background-color: var(--secondary-hover-color);
}
</style>
