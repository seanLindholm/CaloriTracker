<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  foods: {
    type: Array,
    required: true
  }
});

const emits = defineEmits(['update:foods', 'close']);

const recipeName = ref('');
const recipePortions = ref(1);
const recipeItems = ref([]);
const itemSearches = ref({});
const selectedItemIndex = ref(null);
const isSearchFocused = ref(false);
const isClosing = ref(false);

const filteredFoods = computed(() => {
  const search = selectedItemIndex.value !== null ? 
    itemSearches.value[selectedItemIndex.value] || '' : '';
  
  if (!search.trim()) {
    return props.foods;
  }
  const searchLower = search.toLowerCase();
  return props.foods.filter(food =>
    food.name.toLowerCase().includes(searchLower)
  );
});

const selectedRecipeItems = computed(() => {
  return recipeItems.value.filter(item => item.foodId !== null && item.foodId !== undefined);
});

// Calculate total nutrition for the recipe
const totalNutrition = computed(() => {
  const nutritionFields = [
    'calories', 'protein', 'carbohydrates', 'fat', 'salt', 'fibre', 'iron',
    'vitaminK', 'vitaminK1', 'vitaminK2',
    'vitaminB1', 'vitaminB2', 'vitaminB6', 'vitaminB12',
    'vitaminA', 'vitaminC', 'vitaminD', 'vitaminD2', 'vitaminD3', 'vitaminE'
  ];

  const totals = {};
  nutritionFields.forEach(field => {
    totals[field] = 0;
  });

  selectedRecipeItems.value.forEach(item => {
    const food = props.foods.find(f => f.id === item.foodId);
    if (food) {
      const amount = parseFloat(item.amount) || 0;
      // Calculate based on unit
      const multiplier = amount / 100;

      nutritionFields.forEach(field => {
        const value = parseFloat(food[field]) || 0;
        totals[field] += value * multiplier;
      });
    }
  });

  return totals;
});

// Calculate per-portion nutrition
const perPortionNutrition = computed(() => {
  const portions = Math.max(1, parseInt(recipePortions.value) || 1);
  const nutrition = {};

  Object.keys(totalNutrition.value).forEach(field => {
    nutrition[field] = totalNutrition.value[field] / portions;
  });

  return nutrition;
});

function addRecipeItem() {
  recipeItems.value.unshift({
    foodId: null,
    amount: 100
  });
  selectedItemIndex.value = 0;
  itemSearches.value = {
    0: ''
  };
}

function removeRecipeItem(index) {
  recipeItems.value.splice(index, 1);
  const reindexedSearches = {};
  recipeItems.value.forEach((_, itemIndex) => {
    if (Object.hasOwn(itemSearches.value, itemIndex)) {
      reindexedSearches[itemIndex] = itemSearches.value[itemIndex];
    }
  });
  itemSearches.value = reindexedSearches;

  if (selectedItemIndex.value === index) {
    selectedItemIndex.value = null;
  } else if (selectedItemIndex.value !== null && selectedItemIndex.value > index) {
    selectedItemIndex.value -= 1;
  }
}

function selectItemForEditing(index) {
  selectedItemIndex.value = index;
  if (!itemSearches.value[index]) {
    itemSearches.value[index] = '';
  }
}

function selectFoodForItem(itemIndex, food) {
  recipeItems.value[itemIndex].foodId = food.id;
  itemSearches.value[itemIndex] = '';
}

function handleSearchFocus() {
  isSearchFocused.value = true;
}

function handleSearchBlur() {
  setTimeout(() => {
    isSearchFocused.value = false;
  }, 200);
}

function saveRecipe() {
  if (!recipeName.value.trim()) {
    alert('Please enter a recipe name');
    return;
  }

  if (recipeItems.value.length === 0) {
    alert('Please add at least one food item to the recipe');
    return;
  }

  // Get the next ID for the new recipe
  const maxId = Math.max(
    0,
    ...props.foods.map(f => f.id || 0)
  );

  const newRecipe = {
    id: maxId + 1,
    name: recipeName.value.trim(),
    unit: 'portion',
    ...perPortionNutrition.value
  };

  const updatedFoods = [...props.foods, newRecipe];
  emits('update:foods', updatedFoods);

  // Reset form
  recipeName.value = '';
  recipePortions.value = 1;
  recipeItems.value = [];
  itemSearches.value = {};
  selectedItemIndex.value = null;
  
  // Auto-close the dialog after saving
  handleClose();
}

function handleClose() {
  isClosing.value = true;
  setTimeout(() => {
    emits('close');
  }, 300);
}
</script>

<template>
  <div class="recipe-dialog-overlay" :class="{ 'closing': isClosing }">
    <div class="recipe-dialog-content">
      <div class="dialog-header">
        <h2>Build Recipe</h2>
        <button class="close-btn" @click="handleClose">&times;</button>
      </div>

      <!-- Recipe Name and Portions -->
      <div class="recipe-info">
        <div class="form-group">
          <label>Recipe Name:</label>
          <input
            v-model="recipeName"
            type="text"
            placeholder="Enter recipe name..."
            class="search-input"
          />
        </div>
        <div class="form-group">
          <label>Portions the recipe makes:</label>
          <input
            v-model="recipePortions"
            type="number"
            min="1"
            class="search-input"
          />
        </div>
      </div>

       <!-- Nutrition Summary -->
      <div class="nutrition-summary">
        <h3>Per Portion Nutrition:</h3>
        <div class="nutrition-grid">
          <div class="nutrition-item">
            <span class="nutrition-label">Calories:</span>
            <span class="nutrition-value">{{ Math.round(perPortionNutrition.calories) }}</span>
          </div>
          <div class="nutrition-item">
            <span class="nutrition-label">Protein:</span>
            <span class="nutrition-value">{{ Math.round(perPortionNutrition.protein * 10) / 10 }}g</span>
          </div>
          <div class="nutrition-item">
            <span class="nutrition-label">Carbs:</span>
            <span class="nutrition-value">{{ Math.round(perPortionNutrition.carbohydrates * 10) / 10 }}g</span>
          </div>
          <div class="nutrition-item">
            <span class="nutrition-label">Fat:</span>
            <span class="nutrition-value">{{ Math.round(perPortionNutrition.fat * 10) / 10 }}g</span>
          </div>
          <div class="nutrition-item">
            <span class="nutrition-label">Salt:</span>
            <span class="nutrition-value">{{ Math.round(perPortionNutrition.salt * 100) / 100 }}g</span>
          </div>
          <div class="nutrition-item">
            <span class="nutrition-label">Fibre:</span>
            <span class="nutrition-value">{{ Math.round(perPortionNutrition.fibre * 10) / 10 }}g</span>
          </div>
        </div>
      </div>


      <!-- Recipe Items -->
      <div class="recipe-items-container">
        <div class="dialog-buttons">
          <h3>Recipe Items:</h3>
          <div class="button-group">
            <button class="save-btn" @click="saveRecipe">Save</button>
            <button class="cancel-btn" @click="handleClose">Cancel</button>
            <button class="add-item-btn" @click="addRecipeItem">+ items</button>
          </div>
        </div>

        <div v-if="recipeItems.length === 0" class="no-foods">
          No items added. Click "Add Item" to get started.
        </div>

        <div v-else class="recipe-items-list">
          <div v-for="(item, index) in recipeItems" :key="index" class="recipe-item">
            <div class="item-food-select themed-scrollbar">
              <input
                v-model="itemSearches[index]"
                type="text"
                placeholder="Search food..."
                class="search-input"
                :disabled="item.foodId !== null"
                @focus="selectItemForEditing(index); handleSearchFocus()"
                @blur="handleSearchBlur"
              />
              <div class="dropdown-list" v-if="selectedItemIndex === index && isSearchFocused && itemSearches[index]?.trim() && filteredFoods.length > 0">
                <button
                  v-for="food in filteredFoods"
                  :key="food.id"
                  type="button"
                  class="dropdown-option"
                  @mousedown.prevent="selectFoodForItem(index, food)"
                >
                  {{ food.name }}
                </button>
              </div>
            </div>

            <div class="item-details">
              <div class="food-name" v-if="item.foodId">
                {{ props.foods.find(f => f.id === item.foodId)?.name || 'Unknown' }}
              </div>
              <div v-else class="food-name-placeholder">Select a food</div>

              <div class="item-amount">
                <input
                  v-model.number="item.amount"
                  type="number"
                  placeholder="Amount"
                  min="1"
                  step="0.1"
                />
                <span class="unit">{{ props.foods.find(f => f.id === item.foodId)?.unit || 'unit(s)' }}</span>
              </div>
            </div>

            <button class="remove-btn" @click="removeRecipeItem(index)">Remove</button>
          </div>
        </div>
      </div>

     
      <!-- Action Buttons -->
      <div class="dialog-buttons">
     
      </div>
    </div>
  </div>
</template>

<style scoped>
.recipe-dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: var(--overlay-color);
  display: flex;
  align-items: flex-end;
  z-index: 3;
  animation: slideUp 0.3s ease-out forwards;
}

.recipe-dialog-overlay.closing {
  animation: slideDown 0.3s ease-in forwards;
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

.recipe-dialog-content {
  background-color: var(--surface-color);
  width: 100%;
  max-width: 1280px;
  margin: 0 auto;
  padding: 20px;
  border-radius: 8px 8px 0 0;
  display: flex;
  flex-direction: column;
  height: 90vh;
  overflow-y: auto;
}

.dialog-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.dialog-header h2 {
  margin: 0;
  color: var(--text-color);
}

.close-btn {
  font-size: 28px;
}

.recipe-info {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 15px;
  margin-bottom: 25px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-group label {
  color: var(--text-color);

  font-weight: 500;
}

.recipe-items-container {
  margin-bottom: 25px;
}

.items-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.items-header h3 {
  margin: 0;
  color: var(--text-color);
}

.add-item-btn {
  padding: 8px 16px;
  background-color: var(--success-color);
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
 
  font-weight: bold;
}

.add-item-btn:hover {
  background-color: var(--success-hover-color);
}

.recipe-items-list {
  display: flex;
  flex-direction: column;
  margin-top:10px;
  gap: 15px;
}

.recipe-item {
  display: grid;
  grid-template-columns: 1fr 2fr auto;
  gap: 15px;
  align-items: flex-start;
  padding: 15px;
  background-color: var(--surface-alt-color);
  border: 1px solid var(--border-color);
  border-radius: 4px;
}

.item-food-select {
  position: relative;
}

.item-details {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.food-name,
.food-name-placeholder {
  padding: 8px;
  background-color: var(--input-bg);
  border: 1px solid var(--border-color);
  border-radius: 3px;
  color: var(--text-color);

  min-height: 32px;
  display: flex;
  align-items: center;
}

.food-name-placeholder {
  color: var(--text-muted-color);
  font-style: italic;
}

.item-amount {
  display: flex;
  gap: 8px;
  align-items: center;
}

.item-amount input {
  width: 80px;
}

.item-food-select .search-input:disabled {
  background-color: var(--border-color);
  color: var(--text-muted-color);
  cursor: not-allowed;
  opacity: 0.6;
}

.unit {
  color: var(--text-muted-color);
}

.remove-btn {
  padding: 8px 16px;
  background-color: var(--danger-color);
  color: white;
}

.remove-btn:hover {
  background-color: var(--danger-hover-color);
}

.dropdown-list {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background-color: var(--input-bg);
  border: 1px solid var(--border-color);
  border-top: none;
  max-height: 200px;
  overflow-y: auto;
  border-radius: 0 0 4px 4px;
}

.dropdown-option {
  width: 100%;
  padding: 10px;
  background: none;
  text-align: left;
}

.dropdown-option:hover {
  background-color: var(--border-color);
}

.nutrition-summary {
  background-color: var(--surface-alt-color);
  padding: 15px;
  border-radius: 4px;
  border: 1px solid var(--border-color);
  margin-bottom: 20px;
}

.nutrition-summary h3 {
  margin: 0 0 15px 0;
  color: var(--text-color);
}

.nutrition-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 15px;
}

.nutrition-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px;
  background-color: var(--input-bg);
  border-radius: 3px;
}

.nutrition-label {
  color: var(--text-muted-color);
  font-weight: 500;
}

.nutrition-value {
  color: var(--text-color);
  font-weight: bold;
}

.dialog-buttons {
  justify-content: space-between;
  align-items: center;
}

.dialog-buttons h3 {
  margin: 0;
}

.button-group {
  display: flex;
  gap: 10px;
}

.save-btn {
  background-color: var(--success-color);
  color: white;
}

.save-btn:hover {
  background-color: var(--success-hover-color);
}

.cancel-btn {
  background-color: var(--border-color);
  color: var(--text-color);
}

.cancel-btn:hover {
  background-color: var(--surface-alt-color);
}

@media (max-width: 768px) {
  .recipe-info {
    grid-template-columns: 1fr;
  }

  .recipe-item {
    grid-template-columns: 1fr;
  }

  .nutrition-grid {
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  }
}
</style>
