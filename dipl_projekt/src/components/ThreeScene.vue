<template>
    <div class="customizer-layout">
      <div ref="threeCanvas" class="three-container"></div>
  
      <button class="customize-btn" @click="sidebarOpen = !sidebarOpen">
        {{ sidebarOpen ? 'Close' : 'Customize' }}
      </button>
  
      <div class="sidebar" v-if="sidebarOpen">
        <h2>Customize</h2>
        <label>
          Logo Color:
          <input type="color" v-model="selectedColors.logo" @input="() => updateColor('logo')" />
        </label>
        <label>
          Laces Color:
          <input type="color" v-model="selectedColors.laces" @input="() => updateColor('laces')" />
        </label>
        <label>
          Soles Color:
          <input type="color" v-model="selectedColors.soles" @input="() => updateColor('soles')" />
        </label>
        <label>
          Side Soles Color:
          <input type="color" v-model="selectedColors.sideSoles" @input="() => updateColor('sideSoles')" />
        </label>
        <label>
          Back Tab Color:
          <input type="color" v-model="selectedColors.backTab" @input="() => updateColor('backTab')" />
        </label>
        <label>
          Bottom Color:
          <input type="color" v-model="selectedColors.bottom" @input="() => updateColor('bottom')" />
        </label>
        <label>
          Change Shoe Model:
          <select v-model="selectedModel" @change="() => swapShoeModel(selectedModel)">
            <option disabled value="">-- Choose Model --</option>
            <option value="shoe.glb">Default Shoe</option>
            <option value="shoewlogo.glb">Shoe with New Logo</option>
          </select>
        </label>
        <div class="preset-toggle">
          <button @click="presetsOpen = !presetsOpen">
            {{ presetsOpen ? 'Hide Presets' : 'Preset Themes' }}
          </button>
        </div>
        <div class="preset-section">
  <div class="preset-buttons" v-if="presetsOpen">
    <button @click="applyPreset('allBlack')">All Black</button>
    <button @click="applyPreset('blackRed')">Black/Red</button>
  </div>
  <button class="reset-btn" @click="resetModel">Reset</button>
</div>
</div>

    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, nextTick, watch, defineEmits } from 'vue'
  import * as THREE from 'three'
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader'
  import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
  
  const threeCanvas = ref(null)
  const sidebarOpen = ref(false)
  const presetsOpen = ref(false)
  const selectedModel = ref('shoe.glb')
  const selectedColors = ref({
    logo: '#ffffff',
    laces: '#ffffff',
    soles: '#ffffff',
    sideSoles: '#ffffff',
    backTab: '#ffffff',
    bottom: '#ffffff'
  })
  const emit = defineEmits(['update-logo-color'])

watch(() => selectedColors.value.logo, (newColor) => {
  emit('update-logo-color', newColor)
})

  
  let logoMesh = null
  let lacesMesh = null
  let solesMesh = null
  let sideSolesMesh = null
  let sideSolesMesh2 = null
  let backTabMesh = null
  let bottomMesh = null
  let modelGroup = null
  let currentLogo = null
  let camera, controls, scene, renderer
  const originalPosition = new THREE.Vector3()
  const originalRotation = new THREE.Euler()
  const originalScale = new THREE.Vector3()
  
  const updateColor = (part) => {
    const color = new THREE.Color(selectedColors.value[part])
    if (part === 'logo' && logoMesh) logoMesh.material.color = color
    if (part === 'laces' && lacesMesh) lacesMesh.material.color = color
    if (part === 'soles' && solesMesh) solesMesh.material.color = color
    if (part === 'sideSoles') {
      if (sideSolesMesh) sideSolesMesh.material.color = color
      if (sideSolesMesh2) sideSolesMesh2.material.color = color
    }
    if (part === 'backTab' && backTabMesh) backTabMesh.material.color = color
    if (part === 'bottom' && bottomMesh) bottomMesh.material.color = color
  }
  
  const applyStoredColors = () => {
    Object.keys(selectedColors.value).forEach(updateColor)
  }
  
  const applyPreset = (preset) => {
    if (preset === 'allBlack') {
      selectedColors.value = {
        logo: '#000000',
        laces: '#000000',
        soles: '#000000',
        sideSoles: '#000000',
        backTab: '#000000',
        bottom: '#000000'
      }
    } else if (preset === 'blackRed') {
      selectedColors.value = {
        logo: '#000000',
        laces: '#000000',
        soles: '#ff0000',
        sideSoles: '#ff0000',
        backTab: '#000000',
        bottom: '#ff0000'
      }
    }
    applyStoredColors()
  }
  
  const swapShoeModel = (fileName) => {
    if (!fileName) return
    if (modelGroup && modelGroup.parent) {
      modelGroup.parent.remove(modelGroup)
    }
    loadModel(`/models/${fileName}`)
  }
  
  const resetModel = () => {
    selectedColors.value = {
      logo: '#ffffff',
      laces: '#ffffff',
      soles: '#ffffff',
      sideSoles: '#ffffff',
      backTab: '#ffffff',
      bottom: '#ffffff'
    }
    selectedModel.value = 'shoe.glb'
    swapShoeModel(selectedModel.value)
    camera.position.set(0, 1.5, 2)
    controls.target.set(0, 0, 0)
    controls.update()
  }
  
  const loadModel = (path) => {
    const loader = new GLTFLoader()
    loader.load(path, (gltf) => {
      const model = gltf.scene
      const box = new THREE.Box3().setFromObject(model)
      const center = new THREE.Vector3()
      box.getCenter(center)
  
      const pivot = new THREE.Object3D()
      scene.add(pivot)
  
      model.position.sub(center)
      model.scale.set(1.5, 1.5, 1.5)
      model.rotation.y = Math.PI / 2
  
      originalPosition.copy(model.position)
      originalRotation.copy(model.rotation)
      originalScale.copy(model.scale)
  
      logoMesh = null
      lacesMesh = null
      solesMesh = null
      sideSolesMesh = null
      sideSolesMesh2 = null
      backTabMesh = null
      bottomMesh = null
  
      model.traverse((child) => {
        if (child.isMesh) {
          child.castShadow = true
          child.receiveShadow = true
  
          const name = child.name.toLowerCase()
          if (name.includes('object_18')) logoMesh = child
          if (name.includes('object_5') || name.includes('object_9')) lacesMesh = child
          if (name.includes('object_13')) solesMesh = child
          if (name.includes('object_25')) sideSolesMesh = child
          if (name.includes('object_26')) sideSolesMesh2 = child
          if (name.includes('object_11')) backTabMesh = child
          if (name.includes('object_24')) bottomMesh = child
        }
      })
  
      modelGroup = model
      pivot.add(model)
      controls.target.set(0, 0, 0)
      controls.update()
  
      applyStoredColors()
    }, undefined, (error) => {
      console.error('❌ Failed to load model:', error)
    })
  }
  
  onMounted(async () => {
    await nextTick()
  
    scene = new THREE.Scene()
    scene.background = new THREE.Color('#f4f4f4')
  
    const width = threeCanvas.value.clientWidth
    const height = threeCanvas.value.clientHeight
  
    camera = new THREE.PerspectiveCamera(35, width / height, 0.1, 1000)
    camera.position.set(0, 1.5, 2)
  
    renderer = new THREE.WebGLRenderer({ antialias: true })
    renderer.setSize(width, height)
    threeCanvas.value.appendChild(renderer.domElement)
  
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.6)
    scene.add(ambientLight)
  
    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8)
    directionalLight.position.set(5, 10, 7.5)
    scene.add(directionalLight)
  
    controls = new OrbitControls(camera, renderer.domElement)
    controls.enableDamping = true
    controls.dampingFactor = 0.05
  
    loadModel('/models/shoe.glb')
  
    const animate = () => {
      requestAnimationFrame(animate)
      controls.update()
      renderer.render(scene, camera)
    }
    animate()
  
    window.addEventListener('resize', () => {
      const width = threeCanvas.value.clientWidth
      const height = threeCanvas.value.clientHeight
      camera.aspect = width / height
      camera.updateProjectionMatrix()
      renderer.setSize(width, height)
    })
  })
  </script>
  
  <style scoped>
  .customizer-layout {
    display: flex;
    justify-content: flex-start;
    align-items: flex-start;
    gap: 2rem;
    padding: 1rem;
    position: relative;
  }
  
  .three-container {
    width: 60vw;
    height: 600px;
    border-radius: 12px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
    background-color: #f4f4f4;
  }
  
  .customize-btn {
    position: absolute;
    top: 1rem;
    right: 1rem;
    z-index: 10;
    padding: 0.5rem 1rem;
    font-weight: bold;
    border: none;
    background-color: #0077ff;
    color: white;
    border-radius: 6px;
    cursor: pointer;
    transition: background-color 0.2s;
  }
  .customize-btn:hover {
    background-color: #005dc1;
  }
  
  .sidebar {
    min-width: 220px;
    padding: 1rem;
    background: white;
    border-radius: 12px;
    box-shadow: 0 4px 16px rgba(0, 0, 0, 0.1);
  }
  
  .sidebar h2 {
    margin-top: 0;
  }
  
  .sidebar label {
    display: block;
    margin-bottom: 1rem;
    font-weight: 500;
  }
  
  .preset-toggle {
    margin-top: 1rem;
  }
  
  .preset-section {
  margin-top: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

  </style>
  