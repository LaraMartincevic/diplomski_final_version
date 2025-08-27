<template>
  <div class="app">
    <Navbar :logoColor="logoColor" />
    <main class="main-section">
      <ThreeScene @update-logo-color="logoColor = $event" />
    </main>
    <Footer />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import Navbar from './components/Navbar.vue'
import Footer from './components/Footer.vue'
import ThreeScene from './components/ThreeScene.vue'

const logoColor = ref('#ffffff')
</script>


<style>
body {
  margin: 0;
  font-family: 'Segoe UI', sans-serif;
  background-color: #f4f4f4;
}

.main-section {
  min-height: 80vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 2rem;
}

#three-container {
  width: 100%;
  max-width: 1200px;
  height: 600px;
  background-color: #ddd;
  border-radius: 12px;
}
</style>
