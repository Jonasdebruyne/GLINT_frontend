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

const selectedFilter = ref("All");
const partners = ref([]);
const searchTerm = ref("");
const selectedpartners = ref([]);
const isDeleteButtonVisible = computed(() => selectedpartners.value.length > 0);
const isPopupVisible = ref(false);

const fetchpartners = async () => {
  try {
    const token = localStorage.getItem("jwtToken");
    const response = await fetch(`${baseURL}/partners`, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const result = await response.json();
    if (result && result.data && result.data.partners) {
      partners.value = result.data.partners;
    } else {
      console.error("No partners found in the response.");
    }
  } catch (error) {
    console.error("Error fetching partners:", error);
  }
};

const toggleSelection = (partnerId) => {
  const index = selectedpartners.value.indexOf(partnerId);
  if (index === -1) {
    selectedpartners.value.push(partnerId);
  } else {
    selectedpartners.value.splice(index, 1);
  }
};

const toggleSelectAll = (event) => {
  selectedpartners.value = event.target.checked
    ? partners.value.map((partner) => partner._id)
    : [];
};

const deleteSelectedpartners = () => {
  if (selectedpartners.value.length === 0) return;
  showPopup();
};

const confirmDelete = async () => {
  await deletepartners();
  hidePopup();
};

const deletepartners = async () => {
  try {
    for (const id of selectedpartners.value) {
      const response = await fetch(`${baseURL}/partners/${id}`, {
        method: "DELETE",
      });
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
    }
    await fetchpartners();
    selectedpartners.value = [];
  } catch (error) {
    console.error("Error deleting partners:", error);
  }
};

const showPopup = () => {
  isPopupVisible.value = true;
};

const hidePopup = () => {
  isPopupVisible.value = false;
};

const totalpartnersCount = computed(() => partners.value.length);

const filteredpartners = computed(() => {
  if (!partners.value) return [];
  const filteredBySearch = partners.value.filter((partner) => {
    return Object.values(partner).some((value) =>
      String(value).toLowerCase().includes(searchTerm.value.toLowerCase())
    );
  });

  const filteredByType = filteredBySearch.filter(
    (partner) =>
      selectedFilter.value === "All" || partner.package === selectedFilter.value // Adjusted to filter by package
  );

  return filteredByType;
});

onMounted(() => {
  fetchpartners();
  setInterval(() => {
    fetchpartners();
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
        <router-link exact to="/admin/AddNewPartnerPage" class="btn active">
          <p>Add new</p>
          <img src="../../assets/icons/add.svg" alt="icon" />
        </router-link>
        <div
          class="btn display"
          :style="{ visibility: isDeleteButtonVisible ? 'visible' : 'hidden' }"
          @click="showPopup"
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
        <option value="Standard">Standard</option>
        <option value="Pro">Pro</option>
      </select>
    </div>

    <div class="partners">
      <div class="top">
        <input
          type="checkbox"
          @change="toggleSelectAll"
          :checked="
            selectedpartners.length === filteredpartners.length &&
            filteredpartners.length > 0
          "
        />
        <p>Partner ID</p>
        <p>Name</p>
        <p>Contact Email</p>
        <p>Contact Phone</p>
        <p>Package</p>
      </div>

      <div class="table-container">
        <ul v-if="filteredpartners.length" class="list">
          <li v-for="partner in filteredpartners" :key="partner._id">
            <input
              type="checkbox"
              @change="toggleSelection(partner._id)"
              :checked="selectedpartners.includes(partner._id)"
            />
            <router-link
              :to="{ name: 'EditPartner', params: { id: partner._id } }"
            >
              <p>{{ partner._id }}</p>
              <p>{{ partner.name }}</p>
              <p>{{ partner.contact_email }}</p>
              <p>{{ partner.contact_phone }}</p>
              <p>{{ partner.package }}</p>
            </router-link>
          </li>
        </ul>

        <div v-else class="no-partners">
          <p>No partners found matching the selected filters.</p>
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

.partners {
  background-color: #1a1a1a;
  width: 100%;
  border-radius: 8px;
  overflow-x: auto;
}

.partners::-webkit-scrollbar {
  display: none;
}

.partners .top {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 40px;
  padding: 4px 16px;
  background-color: #1d1d1d;
  align-items: center;
}

.partners .list {
  display: flex;
  flex-direction: column;
  padding: 16px;
  gap: 16px;
  max-height: 300px;
}

.partners .list li {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 40px;
  align-items: center;
}

.partners .list li a {
  display: contents; /* Zorg ervoor dat de a-tag zich gedraagt als inhoud van de grid */
}

.partners .top p,
.partners .list li p {
  color: var(--white);
  white-space: nowrap; /* Zorg ervoor dat tekst niet over meerdere regels gaat */
  overflow: hidden; /* Verberg tekst die te lang is */
  text-overflow: ellipsis; /* Voeg een ellipsis toe als de tekst te lang is */
}

.partners .top p:nth-child(1),
.partners .list li p:nth-child(1) {
  width: 200px; /* De Order ID kolom is nu groter */
}

.partners .top p:nth-child(2),
.partners .list li p:nth-child(2) {
  width: 200px;
}

.partners .top p:nth-child(3),
.partners .list li p:nth-child(3) {
  width: 200px;
}

.partners .top p:nth-child(4),
.partners .list li p:nth-child(4) {
  width: 200px;
}

.partners .top p:nth-child(5),
.partners .list li p:nth-child(5) {
  width: 200px;
}

.partners .top p:nth-child(6),
.partners .list li p:nth-child(6) {
  width: 200px;
}

.partners .list li p:hover {
  text-decoration: underline;
}

.no-partners {
  padding: 16px;
  text-align: center;
  color: var(--white);
}
</style>
