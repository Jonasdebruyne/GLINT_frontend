<script setup>
import { ref, onMounted, computed } from "vue";
import Navigation from "../../components/navComponent.vue";
import router from "../../router";

const jwtToken = localStorage.getItem("jwtToken");
if (!jwtToken) {
  router.push("/login");
}

const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

const selectedTypeFilter = ref("All");
const data = ref([]);
const searchTerm = ref("");
const selectedProducts = ref([]);
const isDeleteButtonVisible = computed(() => selectedProducts.value.length > 0);
const isPopupVisible = ref(false);

const fetchData = async () => {
  try {
    const token = localStorage.getItem("jwtToken");
    const response = await fetch(`${baseURL}/products`, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const result = await response.json();
    // Correcte datatoewijzing
    data.value = result.data.products; // hier de wijziging
  } catch (error) {
    console.error("Error fetching data:", error);
  }
};

const toggleSelection = (productId) => {
  const index = selectedProducts.value.indexOf(productId);
  if (index === -1) {
    selectedProducts.value.push(productId);
  } else {
    selectedProducts.value.splice(index, 1);
  }
};

const toggleSelectAll = (event) => {
  selectedProducts.value = event.target.checked
    ? data.value.map((product) => product._id)
    : [];
};

const deleteSelectedProducts = () => {
  if (selectedProducts.value.length === 0) return;
  showPopup();
};

const confirmDelete = async () => {
  await deleteProducts();
  hidePopup();
};

const deleteProducts = async () => {
  try {
    for (const id of selectedProducts.value) {
      const response = await fetch(`${baseURL}/products/${id}`, {
        method: "DELETE",
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
    }

    await fetchData();
    selectedProducts.value = [];
  } catch (error) {
    console.error("Error deleting products:", error);
  }
};

const showPopup = () => {
  isPopupVisible.value = true;
};

const hidePopup = () => {
  isPopupVisible.value = false;
};

onMounted(() => {
  fetchData();
});

const filteredProducts = computed(() => {
  if (!data.value) return [];

  const filteredBySearch = data.value.filter((product) => {
    return Object.values(product).some((value) =>
      String(value).toLowerCase().includes(searchTerm.value.toLowerCase())
    );
  });

  const filteredByType = filteredBySearch.filter(
    (product) =>
      selectedTypeFilter.value === "All" ||
      product.typeOfProduct === selectedTypeFilter.value
  );

  return filteredByType;
});
</script>

<template>
  <Navigation />
  <div class="overlay" v-if="isPopupVisible"></div>
  <div class="content">
    <div class="popup" v-if="isPopupVisible">
      <img src="../../assets/icons/cross-circle.svg" alt="icon" />
      <div class="text">
        <h2>Are you sure?</h2>
        <p>Do you really want to delete this item(s)?</p>
        <div class="btns">
          <button @click="hidePopup">Cancel</button>
          <button @click="confirmDelete" class="btn active">Delete</button>
        </div>
      </div>
    </div>
    <div class="menu">
      <div class="btns">
        <router-link exact to="/admin/add-new-product" class="btn active">
          <p>Add new</p>
          <img src="../../assets/icons/add.svg" alt="icon" />
        </router-link>
        <div
          @click="deleteSelectedProducts"
          class="btn display"
          :style="{ visibility: isDeleteButtonVisible ? 'visible' : 'hidden' }"
        >
          <p>Delete item(s)</p>
        </div>
      </div>

      <div class="search">
        <img src="../../assets/icons/search.svg" alt="icon" />
        <input placeholder="Search" v-model="searchTerm" />
      </div>

      <select class="filter" v-model="selectedTypeFilter">
        <option value="All">All</option>
        <option value="optical">Optical</option>
        <option value="sun">Sun</option>
      </select>
    </div>
    <div class="products">
      <div class="top">
        <input
          type="checkbox"
          @change="toggleSelectAll"
          :checked="
            selectedProducts.length === filteredProducts.length &&
            filteredProducts.length > 0
          "
        />
        <p>Product Code</p>
        <p>Type of product</p>
        <p>Brand</p>
        <p>Product name</p>
        <p>Colors</p>
        <p>Description</p>
        <p>Status</p>
        <p>Glass colour</p>
      </div>

      <ul v-if="filteredProducts.length" class="list">
        <li v-for="product in filteredProducts" :key="product._id">
          <input
            type="checkbox"
            @change="toggleSelection(product._id)"
            :checked="selectedProducts.includes(product._id)"
          />
          <router-link
            :to="{ name: 'EditProduct', params: { id: product._id } }"
          >
            <p>{{ product.productCode }}</p>
            <p>{{ product.typeOfProduct }}</p>
            <p>{{ product.brand }}</p>
            <p>{{ product.productName }}</p>
            <p>{{ product.colors.join(", ") }}</p>
            <!-- Voeg join toe om de kleuren weer te geven -->
            <p>{{ product.description }}</p>
            <p>{{ product.activeUnactive }}</p>
            <p>{{ product.glassColor }}</p>
            <!-- Correcte eigenschap -->
          </router-link>
        </li>
      </ul>

      <div v-else class="no-products">
        <p>Geen producten gevonden voor de huidige zoekopdracht of filter.</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.content {
  width: 100%;
  height: 100vh;
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.6);
  z-index: 999;
}

.popup {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 24px;
  background-color: #1a1a1a;
  padding: 24px 32px;
  border-radius: 8px;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1000;
}

.popup img {
  width: 64px;
  height: 64px;
}

.popup .text {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.popup .text .btns {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.popup .text .btns button {
  color: rgba(255, 255, 255, 0.6);
  border: none;
  background: transparent;
  cursor: pointer;
}

.popup .text .btns .active {
  background-color: #d34848;
  color: var(--white);
}

.menu {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  width: 100%;
}

.menu .btns {
  display: flex;
  flex-direction: row;
  gap: 16px;
  align-items: center;
}

.btn.display {
  visibility: hidden;
  border: 1px solid #d34848;
  background-color: rgba(211, 72, 72, 0.2);
  border-radius: 8px;
  padding: 4px 12px;
}

.btn.display p {
  color: #d34848;
}

.search,
select {
  padding: 4px 12px;
  border-radius: 8px;
  border: none;
  border: 1px solid #1d1d1d;
  background-color: #1d1d1d;
  color: var(--white);
  width: 320px;
}

.no-products {
  text-align: center;
  color: #fff;
  padding: 20px;
}

.search {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 8px;
}

.products {
  background-color: #1a1a1a;
  width: 100%;
  border-radius: 8px;
  overflow-x: auto;
  white-space: nowrap;
}

.products::-webkit-scrollbar {
  display: none;
}

.products .top {
  display: grid;
  grid-template-columns: repeat(10, 1fr);
  gap: 80px;
  border-radius: 8px 8px 0 0;
  padding: 4px 16px;
  background-color: #1d1d1d;
}

.products .list {
  display: flex;
  flex-direction: column;
  padding: 16px;
  gap: 16px;
}

.products .list li {
  display: flex;
  flex-direction: row;
  align-items: center;
}

.products .list li a {
  display: grid;
  grid-template-columns: repeat(10, 1fr);
  gap: 80px;
  padding-left: 80px;
}
.products .list li p {
  color: var(--white);
}

.products .list li p:hover {
  text-decoration: underline;
}

.products .top p:nth-child(2),
.products .list li p:nth-child(1) {
  width: 190px;
}

.products .top p:nth-child(3),
.products .list li p:nth-child(2) {
  width: 150px;
}

.products .top p:nth-child(4),
.products .list li p:nth-child(3) {
  width: 150px;
}

.products .top p:nth-child(5),
.products .list li p:nth-child(6) {
  width: 190px;
}

.products .top p:nth-child(6),
.products .list li p:nth-child(7) {
  width: 180px;
}

.products .top p:nth-child(7),
.products .list li p:nth-child(8) {
  width: 300px;
}

.products .top p:nth-child(8),
.products .list li p:nth-child(9) {
  width: 90px;
}

.products .top p:nth-child(9),
.products .list li p:nth-child(10) {
  width: 100px;
}

.products .top input {
  width: 20px;
}

.search input {
  color: var(--white);
}
</style>
