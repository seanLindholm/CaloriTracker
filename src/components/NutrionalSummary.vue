<script setup>
import { computed, ref } from 'vue';

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

const vitaminsExpanded = ref(false);

const nutritionalStats = computed(() => {
  return props.trackedFoods.reduce((stats, item) => {
    const food = props.foods.find(f => f.id === item.foodId);
    if (food) {
      const parsedUnits = parseInt(item.units ?? item.grams, 10);
      const amount = Math.max(1, Number.isNaN(parsedUnits) ? 1 : parsedUnits);
      const measurement = item.measurement ?? food.unit;
      const multiplier = measurement === 'portion' ? amount : amount / 100;
      stats.protein += food.protein * multiplier;
      stats.carbohydrates += food.carbohydrates * multiplier;
      stats.fibre += food.fibre * multiplier;
      stats.fat += food.fat * multiplier;
      stats.salt += food.salt * multiplier;
      stats.vitaminA += food.vitaminA * multiplier;
      stats.vitaminD += food.vitaminD * multiplier;
      stats.vitaminD3 += food.vitaminD3 * multiplier;
      stats.vitaminD2 += food.vitaminD2 * multiplier;
      stats.vitaminE += food.vitaminE * multiplier;
      stats.vitaminK += food.vitaminK * multiplier;
      stats.vitaminK1 += food.vitaminK1 * multiplier;
      stats.vitaminK2 += food.vitaminK2 * multiplier;
      stats.vitaminB1 += food.vitaminB1 * multiplier;
      stats.vitaminB2 += food.vitaminB2 * multiplier;
      stats.vitaminB6 += food.vitaminB6 * multiplier;
      stats.vitaminB12 += food.vitaminB12 * multiplier;
      stats.vitaminC += food.vitaminC * multiplier;
      stats.iron += food.iron * multiplier;
    }
    return stats;
  }, { 
    protein: 0, carbohydrates: 0, fibre: 0, fat: 0, salt: 0, 
    vitaminA: 0, vitaminD: 0, vitaminD3: 0, vitaminD2: 0, vitaminE: 0,
    vitaminK: 0, vitaminK1: 0, vitaminK2: 0, vitaminB1: 0, vitaminB2: 0,
    vitaminB6: 0, vitaminB12: 0, vitaminC: 0, iron: 0
  });
});
</script>

<template>
  <div class="nutrients-section">
    <h2>Nutritional Summary</h2>
    <div class="nutrients-scroll themed-scrollbar">
      <table class="nutrients-table">
        <thead>
          <tr>
            <th>Nutrient</th>
            <th>Value</th>
            <th>Unit</th>
          </tr>
        </thead>
        <tbody>
        <!-- Macronutrients -->
        <tr>
          <td>Protein</td>
          <td>{{ Math.round(nutritionalStats.protein * 100) / 100 }}</td>
          <td>g</td>
        </tr>
        <tr>
          <td>Carbohydrates</td>
          <td>{{ Math.round(nutritionalStats.carbohydrates * 100) / 100 }}</td>
          <td>g</td>
        </tr>
        <tr>
          <td>Fibre</td>
          <td>{{ Math.round(nutritionalStats.fibre * 100) / 100 }}</td>
          <td>g</td>
        </tr>
        <tr>
          <td>Fat</td>
          <td>{{ Math.round(nutritionalStats.fat * 100) / 100 }}</td>
          <td>g</td>
        </tr>
        <tr>
          <td>Salt</td>
          <td>{{ Math.round(nutritionalStats.salt * 100) / 100 }}</td>
          <td>g</td>
        </tr>
        <tr>
          <td>Iron</td>
          <td>{{ Math.round(nutritionalStats.iron * 100) / 100 }}</td>
          <td>mg</td>
        </tr>

        <!-- Vitamins -->
        <tr
          class="nutrient-category collapsible"
          @click="vitaminsExpanded = !vitaminsExpanded"
        >
          <td colspan="3">
            Vitamins
            <span class="chevron" :class="{ expanded: vitaminsExpanded }"></span>
          </td>
        </tr>
        <template v-if="vitaminsExpanded">
          <tr>
            <td>Vitamin A</td>
            <td>{{ Math.round(nutritionalStats.vitaminA * 100) / 100 }}</td>
            <td>RE (µg)</td>
          </tr>
          <tr>
            <td>Vitamin D</td>
            <td>{{ Math.round(nutritionalStats.vitaminD * 100) / 100 }}</td>
            <td>µg</td>
          </tr>
          <tr>
            <td>Vitamin D3</td>
            <td>{{ Math.round(nutritionalStats.vitaminD3 * 100) / 100 }}</td>
            <td>µg</td>
          </tr>
          <tr>
            <td>Vitamin D2</td>
            <td>{{ Math.round(nutritionalStats.vitaminD2 * 100) / 100 }}</td>
            <td>µg</td>
          </tr>
          <tr>
            <td>Vitamin E</td>
            <td>{{ Math.round(nutritionalStats.vitaminE * 100) / 100 }}</td>
            <td>alfa-TE</td>
          </tr>
          <tr>
            <td>Vitamin K</td>
            <td>{{ Math.round(nutritionalStats.vitaminK * 100) / 100 }}</td>
            <td>µg</td>
          </tr>
          <tr>
            <td>Vitamin K1</td>
            <td>{{ Math.round(nutritionalStats.vitaminK1 * 100) / 100 }}</td>
            <td>µg</td>
          </tr>
          <tr>
            <td>Vitamin K2</td>
            <td>{{ Math.round(nutritionalStats.vitaminK2 * 100) / 100 }}</td>
            <td>µg</td>
          </tr>
          <tr>
            <td>Vitamin B1</td>
            <td>{{ Math.round(nutritionalStats.vitaminB1 * 100) / 100 }}</td>
            <td>mg</td>
          </tr>
          <tr>
            <td>Vitamin B2</td>
            <td>{{ Math.round(nutritionalStats.vitaminB2 * 100) / 100 }}</td>
            <td>mg</td>
          </tr>
          <tr>
            <td>Vitamin B6</td>
            <td>{{ Math.round(nutritionalStats.vitaminB6 * 100) / 100 }}</td>
            <td>mg</td>
          </tr>
          <tr>
            <td>Vitamin B12</td>
            <td>{{ Math.round(nutritionalStats.vitaminB12 * 100) / 100 }}</td>
            <td>µg</td>
          </tr>
          <tr>
            <td>Vitamin C</td>
            <td>{{ Math.round(nutritionalStats.vitaminC * 100) / 100 }}</td>
            <td>mg</td>
          </tr>
        </template>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.nutrients-section {
  padding: 20px;
  max-width: 1280px;
  margin-left: auto;
  margin-right: auto;
}

.nutrients-section h2 {
  font-size: 20px;
  color: var(--text-color);
  margin-bottom: 16px;
}

.nutrients-table tbody tr.nutrient-category {
  background-color: var(--surface-alt-color);
  font-weight: 600;
  color: var(--text-muted-color);
}

.nutrients-table tbody tr.nutrient-category.collapsible {
  cursor: pointer;
  user-select: none;
}

.nutrients-table tbody tr.nutrient-category.collapsible:hover {
  background-color: var(--surface-alt-color);
  opacity: 0.85;
}

.nutrients-table tbody tr.nutrient-category td {
  padding: 10px 16px;
  font-size: 13px;
  letter-spacing: 0.5px;
}

.chevron {
  display: inline-block;
  margin-left: 8px;
  font-size: 16px;
}

.chevron::after {
  content: '+';
}

.chevron.expanded::after {
  content:'-';
}

@media (min-width: 1000px) {
  .nutrients-scroll {
    max-height: var(--summary-scroll-max-height);
    overflow-y: auto;
    scrollbar-gutter: stable;
    border-radius: 6px;
  }
}

@media (max-width: 640px) {
  .nutrients-section {
    padding: 12px;
  }

  .nutrients-table thead th {
    padding: 10px 8px;
    font-size: 13px;
  }

  .nutrients-table tbody td {
    padding: 10px 8px;
    font-size: 14px;
  }
}
</style>
