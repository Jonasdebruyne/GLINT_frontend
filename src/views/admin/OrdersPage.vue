<script lang="ts" setup>
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

const selectedFilter = ref("All");
const orders = ref([]);
const searchTerm = ref("");
const selectedOrders = ref([]);
const isDeleteButtonVisible = computed(() => selectedOrders.value.length > 0);
const isPopupVisible = ref(false);

const fetchOrders = async () => {
  try {
    const token = localStorage.getItem("jwtToken");
    const response = await fetch(`${baseURL}/orders`, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const result = await response.json();
    console.log("Orders fetched:", result); // Controleer de gehele response
    if (result && result.data && result.data.orders) {
      orders.value = result.data.orders;
    } else {
      console.error("No orders found in the response.");
    }
  } catch (error) {
    console.error("Error fetching orders:", error);
  }
};

const toggleSelection = (orderId: string) => {
  const index = selectedOrders.value.indexOf(orderId);
  if (index === -1) {
    selectedOrders.value.push(orderId);
  } else {
    selectedOrders.value.splice(index, 1);
  }
};

const toggleSelectAll = (event: any) => {
  selectedOrders.value = event.target.checked
    ? orders.value.map((order) => order._id)
    : [];
};

const deleteSelectedOrders = () => {
  if (selectedOrders.value.length === 0) return;
  showPopup();
};

const confirmDelete = async () => {
  await deleteOrders();
  hidePopup();
};

const deleteOrders = async () => {
  try {
    for (const id of selectedOrders.value) {
      const response = await fetch(`${baseURL}/orders/${id}`, {
        method: "DELETE",
      });
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
    }

    await fetchOrders();
    selectedOrders.value = [];
  } catch (error) {
    console.error("Error deleting orders:", error);
  }
};

const showPopup = () => {
  isPopupVisible.value = true;
};

const hidePopup = () => {
  isPopupVisible.value = false;
};

const totalOrdersCount = computed(() => orders.value.length);

const filteredOrders = computed(() => {
  if (!orders.value) return [];
  console.log("All orders:", orders.value); // Bekijk alle orders voor de filter
  const filteredBySearch = orders.value.filter((order) => {
    return Object.values(order).some((value) =>
      String(value).toLowerCase().includes(searchTerm.value.toLowerCase())
    );
  });
  console.log("Filtered by search:", filteredBySearch); // Bekijk orders na search filter

  const filteredByType = filteredBySearch.filter(
    (order) =>
      selectedFilter.value === "All" ||
      order.orderStatus === selectedFilter.value
  );
  console.log("Filtered by type:", filteredByType); // Bekijk orders na status filter

  return filteredByType;
});

onMounted(() => {
  fetchOrders();
  setInterval(() => {
    fetchOrders();
  }, 5000);
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
        <p>Aantal orders: {{ totalOrdersCount }}</p>
        <div
          @click="deleteSelectedOrders"
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

      <select class="filter" v-model="selectedFilter">
        <option value="All">All</option>
        <option value="pending">Pending</option>
        <option value="shipped">Shipped</option>
        <option value="delivered">Delivered</option>
        <option value="cancelled">Cancelled</option>
      </select>
    </div>

    <div class="orders">
      <div class="top">
        <input
          type="checkbox"
          @change="toggleSelectAll"
          :checked="
            selectedOrders.length === filteredOrders.length &&
            filteredOrders.length > 0
          "
        />
        <p>Order ID</p>
        <p>ProductCode</p>
        <p>LacesColor</p>
        <p>SoleColor</p>
        <p>InsideColor</p>
        <p>OutsideColor</p>
        <p>Status</p>
      </div>

      <div class="table-container">
        <ul v-if="filteredOrders.length" class="list">
          <li v-for="order in filteredOrders" :key="order._id">
            <input
              type="checkbox"
              @change="toggleSelection(order._id)"
              :checked="selectedOrders.includes(order._id)"
            />
            <router-link :to="{ name: 'EditOrder', params: { id: order._id } }">
              <p>{{ order._id }}</p>
              <p>{{ order.productCode }}</p>
              <p>{{ order.lacesColor }}</p>
              <p>{{ order.soleColor }}</p>
              <p>{{ order.insideColor }}</p>
              <p>{{ order.outsideColor }}</p>
              <p>{{ order.orderStatus }}</p>
            </router-link>
          </li>
        </ul>

        <div v-else class="no-orders">
          <p>No orders found matching the selected filters.</p>
        </div>
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

.popup img {
  width: 64px;
  height: 64px;
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

.search,
select {
  padding: 4px 12px;
  border-radius: 8px;
  border: none;
  border: 1px solid #1d1d1d;
  background-color: #1d1d1d;
  color: var(--white);
  width: 320px;
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 4px;
}

.search input {
  color: var(--white);
}

.orders {
  background-color: #1a1a1a;
  width: 100%;
  border-radius: 8px;
  overflow-x: auto;
}

.orders::-webkit-scrollbar {
  display: none;
}

.orders .top {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 40px;
  padding: 4px 16px;
  background-color: #1d1d1d;
  align-items: center;
}

.orders .list {
  display: flex;
  flex-direction: column;
  padding: 16px;
  gap: 16px;
  max-height: 300px;
}

.orders .list li {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 40px;
  align-items: center;
}

.orders .list li a {
  display: contents; /* Zorg ervoor dat de a-tag zich gedraagt als inhoud van de grid */
}

.orders .top p,
.orders .list li p {
  color: var(--white);
  white-space: nowrap; /* Zorg ervoor dat tekst niet over meerdere regels gaat */
  overflow: hidden; /* Verberg tekst die te lang is */
  text-overflow: ellipsis; /* Voeg een ellipsis toe als de tekst te lang is */
}

.orders .top p:nth-child(1),
.orders .list li p:nth-child(1) {
  width: 150px; /* De Order ID kolom is nu groter */
}

.orders .top p:nth-child(2),
.orders .list li p:nth-child(2) {
  width: 150px;
}

.orders .top p:nth-child(3),
.orders .list li p:nth-child(3) {
  width: 180px;
}

.orders .top p:nth-child(4),
.orders .list li p:nth-child(4) {
  width: 150px;
}

.orders .list li p:hover {
  text-decoration: underline;
}

.no-orders {
  padding: 16px;
  text-align: center;
  color: var(--white);
}
</style>
