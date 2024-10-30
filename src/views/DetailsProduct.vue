<template>
    <div class="glint-customizer">
      <header class="header">
        <h1>GLINT</h1>
        <nav class="steps">
          <span class="step active">1. SHAPE</span>
          <span class="step">2. MATERIAL</span>
          <span class="step">3. COLOUR</span>
          <span class="step">4. OVERVIEW</span>
        </nav>
      </header>
  
      <div class="content-wrapper">
        <div class="config-options">
          <button @click="undo" class="config-button">
            <img src="https://img.icons8.com/ios-glyphs/30/ffffff/undo.png" alt="Undo" />
          </button>
          <button @click="redo" class="config-button">
            <img src="https://img.icons8.com/ios-glyphs/30/ffffff/redo.png" alt="Redo" />
          </button>
        </div>
        <div class="image-container">
          <canvas ref="canvas" class="glasses-canvas"></canvas>
        </div>
        <div class="shape-options-nav">
          <div class="shape-options-grid">
            <div v-for="shape in shapes" :key="shape" class="shape-option" @click="selectShape(shape)" :class="{ selected: selectedShape === shape }">
              <div class="shape-icon" :class="shape.toLowerCase()"></div>
              <div class="shape-label">{{ shape }}</div>
            </div>
          </div>
        </div>
      </div>
  
      <footer class="product-info">
        <div class="product-details">
          <div class="product-name">
            <p>Product name</p>
            <h2 style="color: #AA91DE;">Bertha B602</h2>
          </div>
          <div class="product-price">
            <p>Total</p>
            <h3 style="color: #AA91DE;">&euro; 209,99</h3>
          </div>
        </div>
      </footer>
    </div>
  </template>
  
  <script>
  import * as THREE from 'three';
  import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
  
  export default {
    data() {
      return {
        shapes: ["Square", "Oval", "Circle", "Rectangle"],
        selectedShape: "Square",
        model: null,
      };
    },
    methods: {
      selectShape(shape) {
        this.selectedShape = shape;
      },
      undo() {
        // Logic for undo action
      },
      redo() {
        // Logic for redo action
      },
      initThree() {
        const canvas = this.$refs.canvas;
        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, canvas.clientWidth / canvas.clientHeight, 0.1, 1000);
        const renderer = new THREE.WebGLRenderer({ canvas, alpha: true });
        renderer.setSize(canvas.clientWidth, canvas.clientHeight);
  
        const light = new THREE.DirectionalLight(0xffffff, 1);
        light.position.set(10, 10, 10).normalize();
        scene.add(light);
  
        const loader = new GLTFLoader();
        loader.load('https://threejs.org/examples/models/gltf/DamagedHelmet/glTF/DamagedHelmet.gltf', (gltf) => {
          const model = gltf.scene;
          scene.add(model);
          model.rotation.y = Math.PI;
          this.model = model;
        });
  
        camera.position.z = 3.5;
  
        const animate = () => {
          requestAnimationFrame(animate);
          if (this.model) {
            this.model.rotation.y += 0.01;
          }
          renderer.render(scene, camera);
        };
        animate();
      },
    },
    mounted() {
      this.initThree();
    },
  };
  </script>
  
  <style scoped>
  .glint-customizer {
    width: 100vw;
    height: 100vh;
    display: flex;
    flex-direction: column;
    color: #fff;
    background-color: #000;
    padding: 20px;
    box-sizing: border-box;
    font-family: 'Arial, sans-serif';
  }
  
  .header {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: space-between;
    padding: 0 40px;
    margin-bottom: 40px;
  }
  
  .header h1 {
    font-size: 2.5rem;
    color: #ffffff;
    letter-spacing: 1px;
  }
  
  .steps {
    display: flex;
    gap: 30px;
    margin: 0 auto;
    text-align: center;
  }
  
  .step {
    font-size: 1.2rem;
    color: #888;
    cursor: pointer;
    transition: color 0.3s;
  }
  
  .step:hover,
  .step.active {
    color: #AA91DE;
  }
  
  .content-wrapper {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-grow: 1;
    padding: 0 40px;
  }
  
  .config-options {
    display: flex;
    flex-direction: column;
    gap: 20px;
  }
  
  .config-button {
    background-color: #333;
    border: none;
    padding: 10px;
    border-radius: 50%;
    cursor: pointer;
    transition: background-color 0.3s, transform 0.3s;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .config-button:hover {
    background-color: #AA91DE;
    transform: scale(1.1);
  }
  
  .image-container {
    flex-grow: 2;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .glasses-canvas {
    width: 100%;
    height: 100%;
  }
  
  .shape-options-nav {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
  }
  
  .shape-options-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
  }
  
  .shape-option {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
    border-radius: 10px;
    background-color: #333;
    cursor: pointer;
    transition: 0.3s;
  }
  
  .shape-option:hover {
    background-color: #444;
  }
  
  .shape-option.selected {
    border: 2px solid #AA91DE;
  }
  
  .shape-icon {
    width: 60px;
    height: 60px;
    margin-bottom: 10px;
    border: 2px solid #888;
  }
  
  .shape-icon.square {
    background-color: transparent;
  }
  
  .shape-icon.oval {
    background-color: transparent;
    border-radius: 50%;
  }
  
  .shape-icon.circle {
    background-color: transparent;
    border-radius: 50%;
  }
  
  .shape-icon.rectangle {
    background-color: transparent;
  }
  
  .shape-label {
    color: #888;
    font-size: 1.1rem;
  }
  
  .shape-option.selected .shape-label {
    color: #AA91DE;
  }
  
  .product-info {
    width: 100%;
    padding: 20px 40px;
    box-sizing: border-box;
  }
  
  .product-details {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
  
  .product-name {
    font-size: 1.4rem;
    color: #fff;
  }
  
  .product-price {
    font-size: 1.4rem;
    color: #AA91DE;
  }
  
  h2, h3 {
    margin: 5px 0;
  }
  </style>
  