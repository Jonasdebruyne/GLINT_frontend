<script lang="ts" setup>
import { ref, computed } from 'vue';

interface Product {
  id: number;
  name: string;
  price: string;
  image: string;
  isNew: boolean;
  category: string;
}

const products = ref<Product[]>(Array.from({ length: 12 }, (_, index) => ({
  id: index + 1,
  name: index % 2 === 0 ? 'The Fuzz' : index % 3 === 0 ? 'The Riff' : 'Bertha',
  price: index % 3 === 0 ? '€ 289,-' : '€ 99,-',
  image: `https://picsum.photos/300?random=${Math.random()}`, // Random image from Picsum
  isNew: index % 5 === 0,
  category: index % 2 === 0 ? 'Sun' : 'Optical',
})));

const selectedCategory = ref<string>('All');

const filterCategory = (category: string) => {
  selectedCategory.value = category;
};

const sortedProducts = computed(() => {
  if (selectedCategory.value === 'All') {
    return products.value;
  }
  if (selectedCategory.value === 'New') {
    return products.value.filter(product => product.isNew);
  }
  return products.value.filter(product => product.category === selectedCategory.value);
});
</script>

<template>
  <div class="collection-page">
    <h1>Product Collection</h1>
    <nav class="collection-nav">
      <ul class="nav-list">
        <li @click="filterCategory('All')" :class="['nav-item', { active: selectedCategory === 'All' }]">All</li>
        <li @click="filterCategory('Sun')" :class="['nav-item', { active: selectedCategory === 'Sun' }]">Sun</li>
        <li @click="filterCategory('Optical')" :class="['nav-item', { active: selectedCategory === 'Optical' }]">Optical</li>
        <li @click="filterCategory('New')" :class="['nav-item', { active: selectedCategory === 'New' }]">New Arrivals</li>
      </ul>
    </nav>
    <div class="product-grid">
      <div class="product-card" v-for="product in sortedProducts" :key="product.id">
        <div class="product-image-container">
          <img :src="product.image" :alt="product.name" class="product-image" />
          <span v-if="product.isNew" class="new-badge">New!</span>
        </div>
        <div class="product-info">
          <h3 class="product-name">{{ product.name }}</h3>
          <p class="product-price">{{ product.price }}</p>
          <span class="product-category">{{ product.category }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.collection-page {
  padding: 20px;
  font-family: Arial, sans-serif;
  max-width: 1200px;
  margin: 0 auto;
  color: #fff;
}

.collection-nav {
  display: flex;
  justify-content: center;
  margin-bottom: 30px;
}

.nav-list {
  list-style: none;
  display: flex;
  gap: 30px;
  padding: 0;
  margin: 0;
}

.nav-item {
  font-size: 1.2rem;
  cursor: pointer;
  padding-bottom: 10px;
  position: relative;
  color: #e5e5e5;
  transition: color 0.3s;
}

.nav-item.active::after {
  content: "";
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  height: 2px;
  background-color: #aa91de;
}

.nav-item:hover {
  color: #aa91de;
}

h1 {
  text-align: center;
  font-size: 2.5rem;
  margin-bottom: 20px;
  color: #fff;
}

.product-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 30px;
  justify-content: center;
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
  background: #333;
  padding: 20px;
}

.product-image {
  width: 100%;
  height: auto;
  object-fit: cover;
  border-bottom: 1px solid #444;
  border-radius: 0;
}

.new-badge {
  position: absolute;
  top: 15px;
  left: 15px;
  background-color: #aa91de;
  color: white;
  padding: 5px 10px;
  font-size: 0.8rem;
  border-radius: 4px;
}

.product-info {
  padding: 20px;
}

.product-name {
  font-size: 1.4rem;
  color: #fff;
  margin: 0;
  font-weight: bold;
}

.product-price {
  font-size: 1.1rem;
  color: #aa91de;
  margin-top: 10px;
}

.product-category {
  font-size: 0.9rem;
  color: #888;
  margin-top: 10px;
  display: block;
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
