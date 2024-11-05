<script lang="ts" setup>
import { ref, watch } from "vue";
import { useRoute } from "vue-router";

// Verkrijg de huidige route
const route = useRoute();
const productCode = ref<string | undefined>(undefined);
const productData = ref<{ productName: string; productPrice: number } | null>(
  null
);
const isLoading = ref<boolean>(true);
const error = ref<string | null>(null);
const selectedShape = ref<string>("square"); // Houdt de geselecteerde vorm bij

// Kijkt naar de `productCode` parameter in de route en laadt data op basis daarvan
watch(
  () => route.params.productCode,
  (newCode) => {
    if (newCode) {
      productCode.value = newCode as string;
      fetchProductData(newCode);
    } else {
      console.error("Product code is undefined");
    }
  },
  { immediate: true }
);

// Functie om de productdata op te halen
function fetchProductData(code: string) {
  isLoading.value = true;
  error.value = null;
  fetch(`http://localhost:3000/api/v1/products/${code}`)
    .then((response) => {
      if (!response.ok) {
        throw new Error("Network response was not ok");
      }
      return response.json();
    })
    .then((data) => {
      productData.value = {
        productName: data.data.product.productName,
        productPrice: data.data.product.productPrice,
      };
    })
    .catch((err) => {
      console.error("Er is een fout opgetreden:", err);
      error.value = "Kan productinformatie niet ophalen.";
    })
    .finally(() => {
      isLoading.value = false;
    });
}

// Functie om de geselecteerde vorm in te stellen
function selectShape(shape: string) {
  selectedShape.value = shape;
}
</script>

<template>
  <div class="container">
    <header>
      <h1>GLINT</h1>
      <nav class="menu">
        <span class="menu-item active">1. SHAPE</span>
        <span class="menu-item">2. MATERIAL</span>
        <span class="menu-item">3. COLOUR</span>
        <span class="menu-item">4. OVERVIEW</span>
      </nav>
      <h1 style="visibility: hidden">GLINT</h1>
    </header>

    <div class="content">
      <div class="btns">
        <div>
          <p>Undo</p>
          <img src="../assets/icons/undo.svg" alt="Undo" />
        </div>
        <div>
          <p>Redo</p>
          <img src="../assets/icons/redo.svg" alt="Redo" />
        </div>
      </div>
      <div class="image-container"></div>
      <div class="shape-options">
        <div
          class="shape-option"
          :class="{ selected: selectedShape === 'square' }"
          @click="selectShape('square')"
        >
          <p class="square"></p>
          <p>Square</p>
        </div>
        <div
          class="shape-option"
          :class="{ selected: selectedShape === 'rectangle' }"
          @click="selectShape('rectangle')"
        >
          <p class="rectangle"></p>
          <p>Rectangle</p>
        </div>
        <div
          class="shape-option"
          :class="{ selected: selectedShape === 'oval' }"
          @click="selectShape('oval')"
        >
          <p class="oval"></p>
          <p>Oval</p>
        </div>
        <div
          class="shape-option"
          :class="{ selected: selectedShape === 'circle' }"
          @click="selectShape('circle')"
        >
          <p class="circle"></p>
          <p>Circle</p>
        </div>
      </div>
    </div>

    <footer class="product-info">
      <div class="product-details">
        <div>
          <p>Product name</p>
          <h2 v-if="productData" style="color: #aa91de" class="product-name">
            {{ productData.productName }}
          </h2>
          <p v-else>Loading...</p>
        </div>
        <div>
          <p>Total</p>
          <h3 v-if="productData" style="color: #aa91de" class="product-price">
            &euro; {{ productData.productPrice.toFixed(2) }}
          </h3>
          <p v-else>Loading...</p>
        </div>
      </div>
      <p v-if="error" class="error">{{ error }}</p>
    </footer>
  </div>
</template>

<style scoped>
.container {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  color: #fff;
  background-color: #000;
  padding: 64px;
  box-sizing: border-box;
  font-family: "Arial, sans-serif";
}

header {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  padding: 0 40px;
  margin-bottom: 40px;
}

header h1 {
  font-size: 1.5rem;
  color: #ffffff;
  letter-spacing: 1px;
}

.menu {
  display: flex;
  gap: 30px;
  margin: 0 auto;
  text-align: center;
}

.menu-item {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.4);
  cursor: pointer;
  transition: color 0.3s;
}

.menu-item:hover,
.menu-item.active {
  color: #aa91de;
}

.content {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  flex-grow: 1;
  padding: 0 2rem;
  width: 100%;
}

.btns {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.btns div {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 0.5rem;
}

.btns div img {
  width: 32px;
  height: 32px;
}

.image-container {
  background-image: url("../assets/images/testGlasses.png");
  background-position: center;
  background-repeat: no-repeat;
  background-size: contain;
  width: 100%;
  height: 300px;
}

.shape-options {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.shape-option {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.shape-option .square {
  width: 64px;
  height: 64px;
  border: 4px solid #a2a2a2;
  background-color: transparent;
}

.shape-option .oval {
  width: 82px;
  height: 64px;
  background-color: transparent;
  border: 4px solid #a2a2a2;
  border-radius: 50%;
}

.shape-option .circle {
  width: 64px;
  height: 64px;
  background-color: transparent;
  border: 4px solid #a2a2a2;
  border-radius: 50%;
}

.shape-option .rectangle {
  width: 91px;
  height: 64px;
  border: 4px solid #a2a2a2;
  background-color: transparent;
}

.shape-label {
  color: #888;
  font-size: 1.1rem;
}

.shape-option.selected p {
  border-color: #aa91de;
  color: #aa91de;
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

.product-details div {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  font-size: 1rem;
}

.product-name {
  color: #fff;
}

.product-price {
  color: #aa91de;
  font-size: 2.5rem;
}
</style>
