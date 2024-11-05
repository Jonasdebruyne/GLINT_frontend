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

const activeMenu = ref<string>("shape"); // Houdt het actieve menu bij

// Functie om het actieve menu te selecteren
function selectMenu(menu: string) {
  activeMenu.value = menu;
}

function selectShape(shape: string) {
  selectedShape.value = shape;
}
</script>

<template>
  <div class="container">
    <header>
      <h1>GLINT</h1>
      <nav class="menu">
        <span
          class="menu-item"
          :class="{ active: activeMenu === 'shape' }"
          @click="selectMenu('shape')"
          >1. SHAPE</span
        >
        <span
          class="menu-item"
          :class="{ active: activeMenu === 'material' }"
          @click="selectMenu('material')"
          >2. MATERIAL</span
        >
        <span
          class="menu-item"
          :class="{ active: activeMenu === 'colour' }"
          @click="selectMenu('colour')"
          >3. COLOUR</span
        >
        <span
          class="menu-item"
          :class="{ active: activeMenu === 'overview' }"
          @click="selectMenu('overview')"
          >4. OVERVIEW</span
        >
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
      <div class="shape-options" v-if="activeMenu === 'shape'">
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
      <div
        class="primaryAndSecondaryMaterials"
        v-if="activeMenu === 'material'"
      >
        <div class="primary">
          <h3>Primary material</h3>
          <div class="colors">
            <div class="color">
              <div class="selected">
                <div></div>
              </div>
              <p>Gold</p>
            </div>
            <div class="color">
              <div></div>
              <p>Brown</p>
            </div>
            <div class="color">
              <div></div>
              <p>Gray</p>
            </div>
          </div>
        </div>
        <div class="secondary">
          <h3>Secondary material</h3>
          <div class="colors">
            <div class="color">
              <div class="selected"></div>
              <p>Stainless steel</p>
            </div>
            <div class="color">
              <div></div>
              <p>Brown</p>
            </div>
            <div class="color">
              <div></div>
              <p>Gray</p>
            </div>
          </div>
        </div>
      </div>
      <div class="primaryAndSecondaryColors" v-if="activeMenu === 'colour'">
        <div class="primary">
          <h3>Primary color</h3>
          <div class="colors">
            <div class="color">
              <div class="selected">
                <div></div>
              </div>
              <p>Green</p>
            </div>
            <div class="color">
              <div></div>
              <p>Gray</p>
            </div>
            <div class="color">
              <div></div>
              <p>Red</p>
            </div>
          </div>
        </div>
        <div class="secondary">
          <h3>Secondary color</h3>
          <div class="colors">
            <div class="color">
              <div class="selected"></div>
              <p>Gold</p>
            </div>
            <div class="color">
              <div></div>
              <p>Green</p>
            </div>
            <div class="color">
              <div></div>
              <p>White</p>
            </div>
          </div>
        </div>
      </div>
      <div class="overview" v-if="activeMenu === 'overview'">
        <div class="column">
          <h3>Shape</h3>
          <p>Rectangle</p>
        </div>
        <div class="column">
          <h3>Primary material</h3>
          <div class="color">
            <div></div>
            <p>Gray</p>
          </div>
        </div>
        <div class="column">
          <h3>Secondary material</h3>
          <div class="color">
            <div></div>
            <p>Gray</p>
          </div>
        </div>
        <div class="column">
          <h3>Primary color</h3>
          <div class="color">
            <div></div>
            <p>Gray</p>
          </div>
        </div>
        <div class="column">
          <h3>Secondary color</h3>
          <div class="color">
            <div></div>
            <p>Gray</p>
          </div>
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
  width: 50%;
  height: 300px;
}

.display {
  display: none !important;
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

.primaryAndSecondaryMaterials,
.primaryAndSecondaryColors {
  display: flex;
  flex-direction: column;
  gap: 32px;
}

.primaryAndSecondaryMaterials .primary,
.primaryAndSecondaryMaterials .secondary,
.primaryAndSecondaryColors .primary,
.primaryAndSecondaryColors .secondary {
  display: flex;
  flex-direction: column;
  gap: 24px;
  width: 100%;
}

.primaryAndSecondaryMaterials .primary .colors,
.primaryAndSecondaryMaterials .secondary .colors,
.primaryAndSecondaryColors .primary .colors,
.primaryAndSecondaryColors .secondary .colors {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  gap: 16px;
  width: 100%;
}

.primaryAndSecondaryMaterials .primary .colors .color,
.primaryAndSecondaryMaterials .secondary .colors .color,
.primaryAndSecondaryColors .primary .colors .color,
.primaryAndSecondaryColors .secondary .colors .color,
.overview .color {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.overview .color {
  align-items: flex-start;
}

.primaryAndSecondaryMaterials .primary .colors .color div,
.primaryAndSecondaryMaterials .secondary .colors .color div,
.primaryAndSecondaryColors .primary .colors .color div,
.primaryAndSecondaryColors .secondary .colors .color div,
.overview .color div {
  background-color: var(--purple);
  border-radius: 50%;
  height: 24px;
  width: 24px;
}

.primaryAndSecondaryMaterials .primary .colors .color div.selected,
.primaryAndSecondaryMaterials .secondary .colors .color div.selected,
.primaryAndSecondaryColors .primary .colors .color div.selected,
.primaryAndSecondaryColors .secondary .colors .color div.selected,
.overview .color div.selected {
  position: relative; /* Nodig voor absolute positionering van de pseudo-elementen */
  border-radius: 50%; /* Zorgt voor een ronde vorm */
  overflow: hidden; /* Zorgt ervoor dat de achtergrondkleur binnen de rand blijft */
}

.primaryAndSecondaryMaterials .primary .colors .color div.selected::after,
.primaryAndSecondaryMaterials .secondary .colors .color div.selected::after,
.primaryAndSecondaryColors .primary .colors .color div.selected::after,
.primaryAndSecondaryColors .secondary .colors .color div.selected::after,
.overview .color div.selected::after {
  content: "";
  position: absolute;
  top: 3px; /* Binnenste witte rand */
  left: 3px; /* Binnenste witte rand */
  right: 3px; /* Binnenste witte rand */
  bottom: 3px; /* Binnenste witte rand */
  border: 1px solid var(--white); /* Binnenste witte rand */
  border-radius: 50%; /* Ronde vorm voor de binnenste rand */
  background-color: yellow; /* Achtergrondkleur geel */
}

.colors .color p {
  font-size: 12px;
  width: 100%;
}

.overview {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.overview .column {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
</style>
