<script lang="ts" setup>
import { ref, onMounted } from "vue";

const products = ref([]);
const loading = ref(false); // Voeg de loading variabele toe
const error = ref(null); // Voeg de error variabele toe voor foutafhandeling

const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

const fetchProducts = async () => {
  try {
    loading.value = true; // Zet loading op true bij het starten van het fetch-proces
    console.log("Fetching products from:", `${baseURL}/products/`);
    const response = await fetch(`${baseURL}/products/`);
    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }
    const result = await response.json();
    products.value = result.data.products; // Dit haalt de producten uit de juiste structuur
    error.value = null; // Reset de error indien succesvol
  } catch (error) {
    console.error("Error fetching products:", error.message);
    error.value = "Er is een fout opgetreden bij het ophalen van de producten."; // Zet de error bericht
  } finally {
    loading.value = false; // Zet loading terug op false nadat het fetch-proces is voltooid
  }
};

onMounted(() => {
  fetchProducts();
});
</script>

<template>
  <div class="collection-page">
    <div class="top">
      <h1>Collections</h1>
      <nav class="collection-nav">
        <p class="active">All</p>
        <p>Optical</p>
        <p>Sun</p>
      </nav>
    </div>
    <div class="product-grid">
      <div v-for="product in products" :key="product._id" class="product-card">
        <div class="product-image-container">
          <div
            class="product-image"
            :style="{ backgroundColor: product.colors[0] }"
          ></div>
        </div>
        <div class="product-info">
          <p class="product-name">{{ product.productName }}</p>
          <p class="product-price">{{ product.productCode }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.collection-page {
  padding: 80px 24px 24px 24px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 48px;
  width: 100%;
}

.collection-page .top {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 24px;
}

.collection-nav {
  display: flex;
  gap: 24px;
}

.collection-nav .active {
  color: var(--white);
  padding-bottom: 8px;
  border-bottom: 1px solid var(--white);
}

.product-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 24px;
  justify-content: center;
  width: 100%;
}

.product-card {
  background: #222;
  border-radius: 10px;
  box-shadow: 0px 8px 20px rgba(0, 0, 0, 0.5);
  overflow: hidden;
  text-align: center;
  position: relative;
  transition: transform 0.3s, box-shadow 0.3s;
  width: calc(25% - 30px);
  max-width: 300px;
}

.product-card:hover {
  transform: scale(1.05);
  box-shadow: 0px 12px 25px rgba(0, 0, 0, 0.8);
}

.product-image-container {
  position: relative;
  width: 100%;
  height: 200px; /* Stel een hoogte in voor de afbeelding */
  background: #333;
  padding: 20px;
}

.product-image {
  width: 100%;
  height: 100%;
  border-bottom: 1px solid #444;
  border-radius: 0;
}

.product-info {
  padding: 16px 24px;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.product-price {
  color: #aa91de;
}

@media (max-width: 768px) {
  .product-card {
    width: calc(50% - 20px);
  }
}

@media (max-width: 480px) {
  .product-card {
    width: calc(100% - 20px);
  }
}
</style>
