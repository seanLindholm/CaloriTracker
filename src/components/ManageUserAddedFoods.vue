<script setup>
import { ref, computed } from 'vue';

const props = defineProps({
  foods: {
    type: Array,
    required: true
  },
  trackedFoods: {
    type: Array,
    required: true
  }
});

const emits = defineEmits(['update:foods', 'update:trackedFoods', 'close']);

const searchTerm = ref('');

// Filter foods based on search term
const filteredFoods = computed(() => {
  if (!searchTerm.value.trim()) {
    return props.foods;
  }
  const searchLower = searchTerm.value.toLowerCase();
  return props.foods.filter(food =>
    food.name.toLowerCase().includes(searchLower) ||
    String(food.id).toLowerCase().includes(searchLower)
  );
});

function updateFood(index, field, value) {
  const updatedFoods = [...props.foods];
  const globalIndex = props.foods.findIndex(
    f => f.id === filteredFoods.value[index].id
  );
  
  if (field === 'name') {
    updatedFoods[globalIndex].name = value;
  } else if (field === 'unit') {
    updatedFoods[globalIndex].unit = value;
  } else if (['calories', 'protein', 'carbohydrates', 'fat', 'salt', 'fibre'].includes(field)) {
    updatedFoods[globalIndex][field] = parseFloat(value) || 0;
  }
  
  emits('update:foods', updatedFoods);
}

function removeFood(index) {
  const globalIndex = props.foods.findIndex(
    f => f.id === filteredFoods.value[index].id
  );
  const foodId = props.foods[globalIndex].id;
  const updatedFoods = props.foods.filter((_, idx) => idx !== globalIndex);
  
  // Also remove this food from tracked foods
  const updatedTrackedFoods = props.trackedFoods.filter(item => item.foodId !== foodId);
  
  emits('update:foods', updatedFoods);
  emits('update:trackedFoods', updatedTrackedFoods);
}

function closePanel() {
  emits('close');
}
</script>

<template>
  <div class="user-foods-panel">
    <div class="panel-header">
      <h2>Manage User Added Foods</h2>
      <button class="close-btn" @click="closePanel">&times;</button>
    </div>

    <div class="search-box">
      <input 
        v-model="searchTerm"
        type="text"
        placeholder="Search foods..."
        class="search-input"
      />
    </div>

    <div v-if="filteredFoods.length === 0" class="no-foods">
      <p v-if="searchTerm">No foods match your search</p>
      <p v-else>No user-added foods yet</p>
    </div>

    <div v-else class="table-wrapper themed-scrollbar">
      <table class="user-foods-table">
        <colgroup>
          <col class="col-name" />
          <col class="col-unit" />
          <col class="col-calories" />
          <col class="col-protein" />
          <col class="col-carbs" />
          <col class="col-fat" />
          <col class="col-salt" />
          <col class="col-fibre" />
          <col class="col-action" />
        </colgroup>
        <thead>
          <tr>
            <th>Name</th>
            <th>Unit</th>
            <th>Calories</th>
            <th>Protein</th>
            <th>Carbs</th>
            <th>Fat</th>
            <th>Salt</th>
            <th>Fibre</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(food, index) in filteredFoods" :key="food.id">
            <td data-label="Name">
              <input
                :value="food.name"
                @change="e => updateFood(index, 'name', e.target.value)"
                class="cell-input"
                type="text"
              />
            </td>
            <td data-label="Unit">
              <select
                :value="food.unit"
                @change="e => updateFood(index, 'unit', e.target.value)"
                class="cell-select"
              >
                <option value="gram">gram</option>
                <option value="ml">ml</option>
                <option value="portion">portion</option>
              </select>
            </td>
            <td data-label="Calories">
              <input
                :value="food.calories"
                @change="e => updateFood(index, 'calories', e.target.value)"
                class="cell-input"
                type="number"
                min="0"
                step="0.1"
              />
            </td>
            <td data-label="Protein">
              <input
                :value="food.protein"
                @change="e => updateFood(index, 'protein', e.target.value)"
                class="cell-input"
                type="number"
                min="0"
                step="0.1"
              />
            </td>
            <td data-label="Carbs">
              <input
                :value="food.carbohydrates"
                @change="e => updateFood(index, 'carbohydrates', e.target.value)"
                class="cell-input"
                type="number"
                min="0"
                step="0.1"
              />
            </td>
            <td data-label="Fat">
              <input
                :value="food.fat"
                @change="e => updateFood(index, 'fat', e.target.value)"
                class="cell-input"
                type="number"
                min="0"
                step="0.1"
              />
            </td>
            <td data-label="Salt">
              <input
                :value="food.salt"
                @change="e => updateFood(index, 'salt', e.target.value)"
                class="cell-input"
                type="number"
                min="0"
                step="0.1"
              />
            </td>
            <td data-label="Fibre">
              <input
                :value="food.fibre"
                @change="e => updateFood(index, 'fibre', e.target.value)"
                class="cell-input"
                type="number"
                min="0"
                step="0.1"
              />
            </td>
            <td data-label="Action">
              <button class="delete-btn" @click="removeFood(index)">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.user-foods-panel {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: var(--surface-color);
  z-index: 999;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.user-foods-panel > * {
  width: 100%;
  max-width: 1100px;
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  flex-shrink: 0;
}

.panel-header h2 {
  margin: 0;
  color: var(--text-color);
  font-size: 20px;
}

.close-btn {
  font-size: 24px;
}

.search-box {
  margin-bottom: 20px;
  flex-shrink: 0;
}

.no-foods {
  padding: 30px;
}

.table-wrapper {
  flex: 1;
  overflow-y: auto;
}

.user-foods-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 13px;
  table-layout: fixed;
}

.col-name {
  width: 30%;
}

.col-unit {
  width: 16%;
}

.col-calories,
.col-protein,
.col-carbs,
.col-fat,
.col-salt,
.col-fibre {
  width: 9%;
}

.col-action {
  width: 7%;
}

.user-foods-table thead {
  background-color: var(--table-header-bg);
}

.user-foods-table th {
  padding: 12px;
  text-align: left;
  font-weight: bold;
  color: var(--text-color);
  border-bottom: 2px solid var(--border-color);
}

.user-foods-table td {
  padding: 8px;
  border-bottom: 1px solid var(--border-color);
  color: var(--text-color);
}

.user-foods-table tbody tr:hover {
  background-color: var(--table-row-hover);
}

.cell-input,
.cell-select {
  width: 100%;
  padding: 6px;
  border: 1px solid var(--border-color);
  border-radius: 3px;
  background-color: var(--input-bg);
  color: var(--text-color);
  font-size: 12px;
  box-sizing: border-box;
}

.cell-input:focus,
.cell-select:focus {
  outline: none;
  border-color: var(--text-color);
}

.delete-btn {
  padding: 6px 12px;
  background-color: var(--danger-color);
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  font-size: 11px;
}

.delete-btn:hover {
  background-color: var(--danger-hover-color);
}

@media (min-width:1100px) {
  .user-foods-table {
    border: 1px solid var(--border-color);
  }
}

@media (max-width: 1100px) {
  .user-foods-table colgroup {
    display: none;
  }

  .user-foods-table,
  .user-foods-table thead,
  .user-foods-table tbody,
  .user-foods-table tr,
  .user-foods-table td {
    display: block;
    width: 100%;
  }

  .user-foods-table thead {
    display: none;
  }

  .user-foods-table tbody tr {
    display: block;
    background-color: var(--surface-color);
    border: 1px solid var(--border-color);
    border-radius: 10px;
    margin-bottom: 15px;
    padding: 15px;
  }

  .user-foods-table tbody tr:hover {
    background-color: var(--table-row-hover);
  }

  .user-foods-table td {
    display: grid;
    grid-template-columns: 120px 1fr;
    align-items: center;
    border: none;
    padding: 8px 0;
    border-bottom: none;
  }

  .user-foods-table td:before {
    content: attr(data-label);
    font-weight: bold;
    color: var(--text-color);
    margin-right: 10px;
  }

  .user-foods-table td:last-child {
    grid-template-columns: auto;
  }

  .user-foods-table td:last-child:before {
    display: none;
  }

  .cell-input,
  .cell-select {
    width: 100%;
  }
}
</style>
