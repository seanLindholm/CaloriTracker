<script setup>
import { ref } from 'vue';

const props = defineProps({
  trackedFoods: {
    type: Array,
    required: true
  }
});

const emit = defineEmits(['manage-foods', 'clear-tracked-foods', 'save-tracking', 'show-history', 'about']);

const menuOpen = ref(false);
const trackingsMenuOpen = ref(false);
const supportDialogOpen = ref(false);

function handleManageFoods() {
  emit('manage-foods');
  menuOpen.value = false;
}

function handleClearTrackedFoods() {
  emit('clear-tracked-foods');
  menuOpen.value = false;
}

function toggleTrackingsMenu() {
  trackingsMenuOpen.value = !trackingsMenuOpen.value;
}

function handleSaveTracking() {
  emit('save-tracking');
  trackingsMenuOpen.value = false;
  menuOpen.value = false;
}

function handleShowHistory() {
  emit('show-history');
  trackingsMenuOpen.value = false;
  menuOpen.value = false;
}

function handleAbout() {
  emit('about');
  menuOpen.value = false;
}

function handleSupport() {
  supportDialogOpen.value = true;
  menuOpen.value = false;
}

function closeSupportDialog() {
  supportDialogOpen.value = false;
}

function toggleMenu() {
  menuOpen.value = !menuOpen.value;
  trackingsMenuOpen.value = false;
}

function closeMenu() {
  menuOpen.value = false;
  trackingsMenuOpen.value = false;
}
</script>

<template>
  <div class="menu-container">
    <!-- Desktop Banner Menu (width > 1000px) -->
    <nav class="banner-menu" aria-label="Main navigation">
      <button class="menu-button" @click="handleManageFoods" aria-label="Manage food items">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
          <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path>
          <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path>
        </svg>
        Manage Fooditems
      </button>
      <button class="menu-button" @click="handleClearTrackedFoods" aria-label="Clear today's registrations">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
          <path d="M3 6h18"></path>
          <path d="M8 6V4h8v2"></path>
          <path d="M19 6l-1 14H6L5 6"></path>
          <path d="M10 11v6"></path>
          <path d="M14 11v6"></path>
        </svg>
        Clear Trackedfoods
      </button>
      <div class="menu-item-with-submenu">
        <button class="menu-button" @click="toggleTrackingsMenu" :aria-expanded="trackingsMenuOpen" aria-label="Trackings menu">
          <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
            <path d="M3 17l6-6 4 4 8-8"></path>
            <path d="M14 7h7v7"></path>
          </svg>
          Trackings
          <svg class="chevron-icon" :class="{ open: trackingsMenuOpen }" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
            <polyline points="6 9 12 15 18 9"></polyline>
          </svg>
        </button>
        <div v-if="trackingsMenuOpen" class="submenu">
          <button class="submenu-button" @click="handleSaveTracking" aria-label="Save current tracking">
            Save current
          </button>
          <button class="submenu-button" @click="handleShowHistory" aria-label="See tracking history">
            See history
          </button>
        </div>
      </div>
      <button class="menu-button" @click="handleAbout" aria-label="About">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
          <circle cx="12" cy="12" r="10"></circle>
          <path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"></path>
          <path d="M12 17v.01"></path>
        </svg>
        About
      </button>
      <button class="menu-button" @click="handleSupport" aria-label="Support">
        <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
          <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path>
        </svg>
        Support
      </button>
    </nav>

    <!-- Mobile Hamburger Menu (width <= 1000px) -->
    <div class="hamburger-menu">
      <button 
        class="hamburger-btn" 
        @click="toggleMenu" 
        :aria-expanded="menuOpen"
        aria-label="Toggle navigation menu"
      >
        <svg class="hamburger-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
          <line x1="3" y1="6" x2="21" y2="6"></line>
          <line x1="3" y1="12" x2="21" y2="12"></line>
          <line x1="3" y1="18" x2="21" y2="18"></line>
        </svg>
      </button>

      <!-- Slide-out Menu -->
      <div v-if="menuOpen" class="menu-overlay" @click="closeMenu" aria-hidden="true"></div>

      <nav v-if="menuOpen" class="slide-out-menu" aria-label="Mobile navigation">
          <button class="menu-button" @click="handleManageFoods" aria-label="Manage food items">
            <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
              <path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path>
              <path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path>
            </svg>
            Manage Fooditems
          </button>
          <button class="menu-button" @click="handleClearTrackedFoods" aria-label="Clear today's registrations">
            <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
              <path d="M3 6h18"></path>
              <path d="M8 6V4h8v2"></path>
              <path d="M19 6l-1 14H6L5 6"></path>
              <path d="M10 11v6"></path>
              <path d="M14 11v6"></path>
            </svg>
            Clear Trackedfoods
          </button>
          <div class="menu-item-with-submenu">
            <button class="menu-button" @click="toggleTrackingsMenu" :aria-expanded="trackingsMenuOpen" aria-label="Trackings menu">
              <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
                <path d="M3 17l6-6 4 4 8-8"></path>
                <path d="M14 7h7v7"></path>
              </svg>
              Trackings
              <svg class="chevron-icon" :class="{ open: trackingsMenuOpen }" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
                <polyline points="6 9 12 15 18 9"></polyline>
              </svg>
            </button>
            <div v-if="trackingsMenuOpen" class="submenu mobile-submenu">
              <button class="submenu-button" @click="handleSaveTracking" aria-label="Save current tracking">
                Save current
              </button>
              <button class="submenu-button" @click="handleShowHistory" aria-label="See tracking history">
                See history
              </button>
            </div>
          </div>
          <button class="menu-button" @click="handleAbout" aria-label="About">
            <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <circle cx="12" cy="12" r="10"></circle>
              <path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"></path>
              <path d="M12 17v.01"></path>
            </svg>
            About
          </button>
          <button class="menu-button" @click="handleSupport" aria-label="Support">
            <svg class="icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
              <path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"></path>
            </svg>
            Support
          </button>
        </nav>
    </div>

    <div
      v-if="supportDialogOpen"
      class="support-dialog-overlay"
      @click.self="closeSupportDialog"
      role="dialog"
      aria-modal="true"
      aria-labelledby="support-dialog-title"
    >
      <div class="support-dialog">
        <h3 id="support-dialog-title">Support</h3>
        <p>If you want to support this and other projects, you can do this in two ways.</p>
        <p>Spend your time watching a short add, do it as much as you would like, we will never force adds on you.</p>
        <button type="button" class="support-action-btn">Watch short add</button>
        <p>Donate directly using github sponsor.</p>
        <button type="button" class="support-action-btn">Donate with GitHub Sponsor</button>
        <p>Anyways thanks for your support, and glad that you like it</p>
        <button type="button" class="support-close-btn" @click="closeSupportDialog">Close</button>
      </div>
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
  z-index: 1000;
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
  background-color: transparent;
  white-space: nowrap;
}

.banner-menu .menu-button {
  padding: 8px 14px;
  border-radius: 6px;
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

.chevron-icon {
  width: 16px;
  height: 16px;
  margin-left: auto;
  transition: transform 0.2s;
}

.chevron-icon.open {
  transform: rotate(180deg);
}

.menu-item-with-submenu {
  position: relative;
}

.submenu {
  position: absolute;
  top: 100%;
  left: 0;
  background-color: var(--surface-color);
  border: 1px solid var(--border-color);
  border-radius: 6px;
  box-shadow: 0 4px 12px var(--shadow-color);
  min-width: 160px;
  z-index: 10;
  margin-top: 4px;
}

.submenu-button {
  display: block;
  width: 100%;
  padding: 10px 16px;
  background: transparent;
  border: none;
  text-align: left;
  color: var(--text-color);
  cursor: pointer;
  font-size: 14px;
  white-space: nowrap;
}

.submenu-button:hover {
  background-color: var(--surface-alt-color);
}

.submenu-button:first-child {
  border-radius: 6px 6px 0 0;
}

.submenu-button:last-child {
  border-radius: 0 0 6px 6px;
}

.mobile-submenu {
  position: static;
  margin-top: 0;
  border: none;
  border-radius: 0;
  box-shadow: none;
  background-color: var(--surface-alt-color);
  padding-left: 20px;
}

.mobile-submenu .submenu-button {
  padding: 8px 16px;
  font-size: 13px;
}

.mobile-submenu .submenu-button:first-child,
.mobile-submenu .submenu-button:last-child {
  border-radius: 0;
}

.support-dialog-overlay {
  position: fixed;
  inset: 0;
  background-color: var(--overlay-color);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
  z-index: 1200;
}

.support-dialog {
  width: min(560px, 100%);
  background-color: var(--surface-color);
  border: 1px solid var(--border-color);
  border-radius: 12px;
  box-shadow: 0 10px 30px var(--shadow-color);
  padding: 20px;
}

.support-dialog h3 {
  margin-top: 0;
}

.support-dialog p {
  margin: 0 0 12px 0;
  line-height: 1.4;
}

.support-action-btn {
  width: 100%;
  margin-bottom: 14px;
  padding: 10px 12px;
  border-radius: 8px;
  background-color: var(--surface-alt-color);
}

.support-action-btn:hover {
  background-color: var(--table-row-hover);
}

.support-close-btn {
  margin-top: 6px;
  width: 100%;
  padding: 10px 12px;
  border-radius: 8px;
  background-color: var(--secondary-color);
  color: white;
}

.support-close-btn:hover {
  background-color: var(--secondary-hover-color);
}
</style>
