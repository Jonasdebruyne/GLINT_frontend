<script setup>
import { ref, reactive, onMounted, computed, watch, provide } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import Navigation from "../../components/navComponent.vue";

// Router setup
const router = useRouter();

// Reactive user object to store user details (gebruik reactive voor betere reactiviteit)
const user = reactive({
  firstName: "",
  lastName: "",
  email: "",
  newEmail: "",
  oldEmail: "",
  password: "",
  newPassword: "",
  oldPassword: "",
  newPasswordRepeat: "",
  country: "",
  city: "",
  postalCode: "",
  profilePicture: "",
  bio: "",
  role: "",
  activeUnactive: true,
});

// Authentication and token handling
const token = localStorage.getItem("jwtToken");
if (!token) {
  router.push("/login");
}

const parseJwt = (token) => {
  try {
    const base64Url = token.split(".")[1];
    const base64 = base64Url.replace(/-/g, "+").replace(/_/g, "/");
    const jsonPayload = decodeURIComponent(
      atob(base64)
        .split("")
        .map((c) => "%" + ("00" + c.charCodeAt(0).toString(16)).slice(-2))
        .join("")
    );
    return JSON.parse(jsonPayload);
  } catch (error) {
    console.error("Error parsing JWT:", error);
    return null;
  }
};

const tokenPayload = parseJwt(token);
const userId = tokenPayload?.userId;
const partnerId = tokenPayload?.partnerId || null;

if (!userId) {
  router.push("/login");
}

// Base URL for API calls
const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

// Partner related data
const partnerPackage = ref(null);

// Fetch user profile data
const fetchUserProfile = async () => {
  try {
    const response = await axios.get(`${baseURL}/users/${userId}`, {
      headers: { Authorization: `Bearer ${token}` },
    });

    const userData = response.data?.data?.user || {};
    // Update the user object
    user.firstName = userData.firstname || "";
    user.lastName = userData.lastname || "";
    user.email = userData.email || "";
    user.oldEmail = userData.email || "";
    user.country = userData.country || "";
    user.city = userData.city || "";
    user.postalCode = userData.postalCode || "";
    user.profilePicture = userData.profilePicture || "";
    user.bio = userData.bio || "";
    user.role = userData.role || "";
    user.activeUnactive = userData.activeUnactive ?? true;
  } catch (error) {
    console.error("Error fetching user profile:", error);
  }
};

// Fetch partner data (if applicable)
const fetchPartnerData = async () => {
  if (!partnerId) return;

  try {
    const response = await axios.get(`${baseURL}/partners/${partnerId}`, {
      headers: { Authorization: `Bearer ${token}` },
    });

    const partner = response.data?.data?.partner || {};
    partnerPackage.value = partner.package || "No package available";
  } catch (error) {
    console.error("Error fetching partner data:", error);
    partnerPackage.value = "Error loading partner data";
  }
};

// Fetch initial data on mount
onMounted(async () => {
  await fetchUserProfile();
  await fetchPartnerData();
});

// Provide the user data to all components (including Navigation)
provide("user", user); // Makes user data available to child components like Navigation

// Watch for changes in user data and update the Navigation component
watch(
  user,
  (newUser) => {
    console.log("User data updated:", newUser);
  },
  { deep: true }
);

// Reactieve data-referenties
const data = ref([]); // Zorg ervoor dat data altijd een lege array is
const selectedUsers = ref([]); // Dit is een ref voor de geselecteerde gebruikers

// Ophalen van gebruikersgegevens
const fetchData = async () => {
  try {
    const token = localStorage.getItem("jwtToken");
    const decodedToken = parseJwt(token); // Decode the token here
    if (!decodedToken) {
      throw new Error("Invalid token or failed to decode token");
    }

    const response = await fetch(`${baseURL}/users`, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });

    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
    const result = await response.json();

    const userRole = decodedToken?.role; // Now you can safely use decodedToken

    // If the user is a platform_admin, show all users, otherwise filter by partnerId
    if (userRole === "platform_admin") {
      data.value = result.data.users;
    } else {
      data.value = result.data.users.filter(
        (user) => user.partnerId === decodedToken?.partnerId // Use partnerId from decodedToken
      );
    }
  } catch (error) {
    console.error("Error fetching data:", error);
  }
};

// Filter de gebruikers op basis van zoekterm en filter
const filteredUsers = computed(() => {
  // Controleer eerst of data.value gedefinieerd is
  if (!data.value) return [];

  return data.value.filter((user) => {
    const matchesSearchTerm =
      user.firstname.toLowerCase().includes(searchTerm.value.toLowerCase()) ||
      user.lastname.toLowerCase().includes(searchTerm.value.toLowerCase()) ||
      user.email.toLowerCase().includes(searchTerm.value.toLowerCase());

    const matchesFilter =
      selectedFilter.value === "All" || user.role === selectedFilter.value;

    return matchesSearchTerm && matchesFilter;
  });
});

// Initialiseer component en haal data op
onMounted(fetchData);

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
    if (result && result.data && result.data.orders) {
      orders.value = result.data.orders;
    } else {
      console.error("No orders found in the response.");
    }
  } catch (error) {
    console.error("Error fetching orders:", error);
  }
};

const toggleSelection = (orderId) => {
  const index = selectedOrders.indexOf(orderId);
  if (index === -1) {
    selectedOrders.push(orderId);
  } else {
    selectedOrders.splice(index, 1);
  }
};

const toggleSelectAll = (event) => {
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
  const filteredBySearch = orders.value.filter((order) => {
    return Object.values(order).some((value) =>
      String(value).toLowerCase().includes(searchTerm.value.toLowerCase())
    );
  });

  const filteredByType = filteredBySearch.filter(
    (order) =>
      selectedFilter.value === "All" ||
      order.orderStatus === selectedFilter.value
  );

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
