<script setup>
import { ref } from 'vue';

const props = defineProps({
  trackedFoods: {
    type: Array,
    required: true
  }
});

const emit = defineEmits(['manage-foods', 'clear-tracked-foods']);

const menuOpen = ref(false);

function handleManageFoods() {
  emit('manage-foods');
  menuOpen.value = false;
}

function handleClearTrackedFoods() {
  emit('clear-tracked-foods');
  menuOpen.value = false;
}

function handleSaveDate() {
  // Save current state with date
  const currentDate = new Date().toISOString();
  localStorage.setItem('lastSaveDate', currentDate);
  alert('Date saved successfully!');
  menuOpen.value = false;
}

function handleAbout() {
  alert('Calorie Tracker v1.0\n\nA simple tool to track your daily calorie intake.');
  menuOpen.value = false;
}

function handleSupport() {
  alert('For support, please visit our website or contact us at support@calorietracker.com');
  menuOpen.value = false;
}

function toggleMenu() {
  menuOpen.value = !menuOpen.value;
}

function closeMenu() {
  menuOpen.value = false;
}
</script>

<template>
  <div class="menu-container">
    <!-- Desktop Banner Menu (width > 1000px) -->
    <nav class="banner-menu">
      <button class="menu-button" @click="handleManageFoods" title="Manage food items">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path>
          <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path>
        </svg>
        Manage Fooditems
      </button>
      <button class="menu-button" @click="handleClearTrackedFoods" title="Clear today's registrations">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M3 6h18"></path>
          <path d="M8 6V4h8v2"></path>
          <path d="M19 6l-1 14H6L5 6"></path>
          <path d="M10 11v6"></path>
          <path d="M14 11v6"></path>
        </svg>
        Clear Trackedfoods
      </button>
      <button class="menu-button" @click="handleSaveDate" title="Save date">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2z"></path>
          <rect x="7" y="3" width="10" height="4"></rect>
          <path d="M7 12h3v5H7z"></path>
        </svg>
        Save date
      </button>
      <button class="menu-button" @click="handleAbout" title="About">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="12" cy="12" r="10"></circle>
          <path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"></path>
          <path d="M12 17v.01"></path>
        </svg>
        About
      </button>
      <button class="menu-button" @click="handleSupport" title="Support">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path>
        </svg>
        Support
      </button>
    </nav>

    <!-- Mobile Hamburger Menu (width <= 1000px) -->
    <div class="hamburger-menu">
      <button class="hamburger-btn" @click="toggleMenu" title="Open menu">
        <svg class="hamburger-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <line x1="3" y1="6" x2="21" y2="6"></line>
          <line x1="3" y1="12" x2="21" y2="12"></line>
          <line x1="3" y1="18" x2="21" y2="18"></line>
        </svg>
      </button>

      <!-- Slide-out Menu -->
      <div v-if="menuOpen" class="menu-overlay" @click="closeMenu"></div>

      <nav v-if="menuOpen" class="slide-out-menu">
          <button class="menu-button" @click="handleManageFoods">
            <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path>
              <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path>
            </svg>
            Manage Fooditems
          </button>
          <button class="menu-button" @click="handleClearTrackedFoods">
            <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M3 6h18"></path>
              <path d="M8 6V4h8v2"></path>
              <path d="M19 6l-1 14H6L5 6"></path>
              <path d="M10 11v6"></path>
              <path d="M14 11v6"></path>
            </svg>
            Clear Trackedfoods
          </button>
          <button class="menu-button" @click="handleSaveDate">
            <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2z"></path>
              <rect x="7" y="3" width="10" height="4"></rect>
              <path d="M7 12h3v5H7z"></path>
            </svg>
            Save date
          </button>
          <button class="menu-button" @click="handleAbout">
            <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <circle cx="12" cy="12" r="10"></circle>
              <path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"></path>
              <path d="M12 17v.01"></path>
            </svg>
            About
          </button>
          <button class="menu-button" @click="handleSupport">
            <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path>
            </svg>
            Support
          </button>
        </nav>
    </div>
  </div>
</template>

<style scoped>
.menu-container {
  width: 100%;
}

/* Desktop Banner Menu */
.banner-menu {
  display: none;
  gap: 8px;
  padding: 12px 20px;
  background-color: var(--surface-color);
  border-bottom: 1px solid var(--border-color);
  flex-wrap: wrap;
  justify-content: center;
}

@media (min-width: 1000px) {
  .banner-menu {
    display: flex;
  }

  .hamburger-menu {
    display: none;
  }
}

/* Mobile Hamburger Menu */
.hamburger-menu {
  display: flex;
  align-items: center;
  position: relative;
}

@media (min-width: 1000px) {
  .hamburger-menu {
    display: none;
  }
}

.hamburger-btn {
    display: flex;
    justify-content: center;
    align-content: center;
    position: fixed;
    top: 10px;
    left: 10px;
    padding: 10px;
    border-radius: 40px;
    background-color: transparent;
    cursor: pointer;
}

.hamburger-btn:hover {
  background-color: var(--surface-alt-color);
}

.hamburger-icon {
  width: 24px;
  height: 24px;
  stroke: var(--text-color);
}

/* Menu Overlay */
.menu-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: var(--overlay-color);
  z-index: 2;
}

/* Slide-out Menu */
.slide-out-menu {
  position: fixed;
  top: 0;
  left: 0;
  width: 250px;
  height: 100vh;
  background-color: var(--surface-color);
  display: flex;
  flex-direction: column;
  gap: 0;
  padding: 60px 0 20px 0;
  z-index: 2;
  box-shadow: 2px 0 8px var(--shadow-color);
  overflow-y: auto;
}

/* Menu Buttons */
.menu-button {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 20px;
  border: none;
  background-color: transparent;
  cursor: pointer;
  color: var(--text-color);
  white-space: nowrap;
}


.banner-menu .menu-button:hover {
  background-color: var(--surface-alt-color);
}

.slide-out-menu .menu-button {
  width: 100%;
  border-radius: 0;
  justify-content: flex-start;
}

.slide-out-menu .menu-button:hover {
  background-color: var(--surface-alt-color);
}

.icon {
  width: 18px;
  height: 18px;
  flex-shrink: 0;
  color: var(--text-color);
}


</style>
