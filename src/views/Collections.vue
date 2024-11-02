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
    <div class="products">
      <div class="row">
        <h2>Optical</h2>
        <p><span>13 </span>items</p>
      </div>
      <div class="product-grid">
        <div
          v-for="product in products"
          :key="product._id"
          :class="['product-card', product.typeOfProduct]"
        >
          <div class="product-image-container">
            <div
              class="product-image"
              :style="{ backgroundColor: product.colors[0] }"
            ></div>
          </div>
          <div class="product-info">
            <p class="product-name">{{ product.productName }}</p>
            <p class="product-price">€ {{ product.productPrice }},00</p>
          </div>
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
  gap: 80px;
}

.collection-nav .active {
  color: var(--white);
  padding-bottom: 8px;
  border-bottom: 1px solid var(--white);
}

.products {
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 24px;
  width: 100%;
  padding: 0 200px;
}

.products .row {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  padding-bottom: 1rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.products .product-grid {
  display: grid;
  grid-template-columns: repeat(1, 1fr);
  justify-content: center;
  gap: 24px;
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
  width: 100%;
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

@media (min-width: 800px) {
  .products .product-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
@media (min-width: 1200px) {
  .products {
    padding: 0 200px;
  }

  .products .product-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}

@media (min-width: 2000px) {
  .products {
    padding: 0 600px;
  }
}
</style>
