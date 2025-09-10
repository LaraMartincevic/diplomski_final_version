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

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

body {
  margin: 0;
  font-family: 'Segoe UI', sans-serif;
  background-color: #f4f4f4;
}

.main-section {
  flex: 1;

  /* center your 3D container as before */
  display: flex;
  align-items: center;
  justify-content: center;

  /* wallpaper behind the 3D area only */
  background-image: url('/wallpaper.jpg'); /* file in /public */
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;

  /* avoid extra whitespace that can push things */
  padding: 0;
  margin: 0;
}

#three-container {
  width: 100%;
  max-width: 1200px;
  height: 600px;
  background-color: #ddd;
  border-radius: 12px;
}
</style>
