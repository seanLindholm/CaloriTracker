<template>
  <div class="about-overlay" role="dialog" aria-modal="true" aria-labelledby="about-title" @click.self="handleClose">
    <div class="about-container">
      <header class="about-header">
        <h2 id="about-title">About</h2>
        <button class="close-btn" @click="handleClose" aria-label="Close dialog">&times;</button>
      </header>
      
      <div class="about-content">
        <section class="about-section">
          <h3>How it works</h3>
          <p>
            This app saves the state in your local browser, essentially that means that every device is a new user, and can be used completely offline.
          </p>
          <p>
            It is built as a progressive web app (PWA) such that you can add it to your home page, then it will live as an app on your phone. Look up guides on how to do this.
          </p>
        </section>

        <section class="about-section">
          <h3>Why have I built this</h3>
          <p>
            Software is meant to be shared, and to help people. And as a person tired of enshittification and paywalls, I have made this open source project. Hopefully this is just one of many, where I will try and make useful small apps that you can use, totally for free! Of course if you feel like it, head over to the support page, and find out how you can help keep this project afloat.
          </p>
        </section>
      </div>

      <div class="about-footer">
        <button class="cancel-btn" @click="handleClose">Close</button>
      </div>
    </div>
  </div>
</template>

<script setup>
const emit = defineEmits(['close']);

function handleClose() {
  emit('close');
}
</script>

<style scoped>
.about-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: var(--overlay-color);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.about-container {
  background-color: var(--bg-color);
  border-radius: 12px;
  width: 90%;
  max-width: 700px;
  box-shadow: 0 4px 20px var(--shadow-color);
  display: flex;
  flex-direction: column;
  max-height: 90vh;
}

.about-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px 24px;
  border-bottom: 1px solid var(--border-color);
  flex-shrink: 0;
}

.about-header h2 {
  margin: 0;
  font-size: 24px;
  color: var(--text-color);
}

.about-content {
  padding: 24px;
  overflow-y: auto;
  max-height: 60vh;
  flex: 1;
}

.about-section {
  margin-bottom: 24px;
}

.about-section:last-child {
  margin-bottom: 0;
}

.about-section h3 {
  margin: 0 0 12px 0;
  font-size: 18px;
  color: var(--primary-color);
  font-weight: 600;
}

.about-section p {
  margin: 0 0 12px 0;
  line-height: 1.6;
  color: var(--text-color);
}

.about-section p:last-child {
  margin-bottom: 0;
}

.about-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  padding: 20px 24px;
  border-top: 1px solid var(--border-color);
  flex-shrink: 0;
}

@media (max-width: 768px) {
  .about-container {
    width: 95%;
    max-height: 95vh;
  }

  .about-content {
    padding: 16px;
  }

  .about-header {
    padding: 16px;
  }

  .about-footer {
    padding: 16px;
  }
}
</style>
