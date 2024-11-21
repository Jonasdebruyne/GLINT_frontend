<script setup>
import { ref, onMounted, computed } from "vue";
import Navigation from "../../components/navComponent.vue";
import router from "../../router";

// JWT-token controle
const jwtToken = localStorage.getItem("jwtToken");
if (!jwtToken) {
  router.push("/login");
}

// API-basispad configureren
const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

// Reactieve data-referenties
const data = ref([]);
const searchTerm = ref("");
const selectedFilter = ref("All");
let selectedUsers = [];
const selectedUser = ref(null); // Toegevoegd voor gedetailleerde weergave
const isPopupVisible = ref(false);

// Computed properties voor UI-staat
const isDeleteButtonVisible = computed(() => selectedUsers.value.length > 0);
const emptyState = computed(() => data.value.length === 0);

// Ophalen van gebruikersgegevens
const fetchData = async () => {
  try {
    const token = localStorage.getItem("jwtToken");
    const response = await fetch(`${baseURL}/users`, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
    const result = await response.json();
    data.value = result.data.users;
  } catch (error) {
    console.error("Error fetching data:", error);
  }
};

// Filter de gebruikers op basis van zoekterm en filter
const filteredUsers = computed(() => {
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

function selectAllUsers(isSelected) {
  selectedUsers = isSelected
    ? filteredUsers
        .map((user) =>
          user._id === "67215000c9333e3c48f10a5d" ? user._id : null
        )
        .filter((id) => id) // Verwijdert de `null` waarden
    : [];
}

// Schakel selectie voor een enkele gebruiker
function toggleSelection(userId) {
  const index = selectedUsers.indexOf(userId);
  if (index === -1) {
    selectedUsers.push(userId);
  } else {
    selectedUsers.splice(index, 1);
  }
}

// Verwijderen van geselecteerde gebruikers
const deleteSelectedUsers = async () => {
  if (!selectedUsers.value.length) return;

  try {
    await Promise.all(
      selectedUsers.value.map(async (userId) => {
        const response = await fetch(`${baseURL}/users/${userId}`, {
          method: "DELETE",
          headers: {
            Authorization: `Bearer ${jwtToken}`,
          },
        });
        if (!response.ok)
          throw new Error(`HTTP error! status: ${response.status}`);
      })
    );

    // Gegevens verversen en selectie wissen na succesvolle verwijdering
    await fetchData();
    selectedUsers.value = [];
  } catch (error) {
    console.error("Error deleting users:", error);
  }
};

// Popup-weergavebeheer
const showPopup = () => (isPopupVisible.value = true);
const hidePopup = () => (isPopupVisible.value = false);
const confirmDelete = async () => {
  await deleteSelectedUsers();
  hidePopup();
};

// Initialiseer component en haal data op
onMounted(fetchData);
</script>

<template>
  <Navigation />
  <div class="overlay" v-if="isPopupVisible"></div>
  <div class="content">
    <div class="popup" v-if="isPopupVisible">
      <img src="../../assets/icons/cross-circle.svg" alt="icon" />
      <div class="text">
        <h2>Are you sure?</h2>
        <p>Do you really want to delete this user?</p>
        <div class="btns">
          <button @click="hidePopup">Cancel</button>
          <button @click="confirmDelete" class="btn active">Delete</button>
        </div>
      </div>
    </div>
    <div class="menu">
      <div class="btns">
        <router-link exact to="/admin/AddNewUserPage" class="btn active">
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
        <option value="admin">Admin</option>
        <option value="user">User</option>
      </select>
    </div>

    <div class="users">
      <div class="top">
        <input
          type="checkbox"
          @change="selectAllUsers($event.target.checked)"
        />
        <p>First name</p>
        <p>Last name</p>
        <p>Email</p>
        <p>Role</p>
        <p>Status</p>
      </div>
      <ul v-if="filteredUsers && filteredUsers.length > 0" class="list">
        <li v-for="user in filteredUsers" :key="user._id">
          <router-link
            :to="{ name: 'EditUser', params: { id: user._id } }"
            class="user-link"
          >
            <input
              type="checkbox"
              @change="toggleSelection(user._id)"
              :checked="selectedUsers.includes(user._id)"
              @click.stop
            />
            <p>{{ user.firstname }}</p>
            <p>{{ user.lastname }}</p>
            <p>{{ user.email }}</p>
            <p>{{ user.role }}</p>
            <p>{{ user.activeUnactive }}</p>
          </router-link>
        </li>
      </ul>
      <div v-else-if="emptyState" class="empty-state">
        <p>No users found matching the selected filters.</p>
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

.search {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 8px;
}

.search input {
  color: var(--white);
}

.users {
  background-color: #1a1a1a;
  width: 100%;
  border-radius: 8px;
  overflow-x: auto;
  white-space: nowrap;
}

.users::-webkit-scrollbar {
  display: none;
}

.users .top {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  align-items: center;
  border-radius: 8px 8px 0 0;
  padding: 4px 16px;
  background-color: #1d1d1d;
}

.users .list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.users .list li {
  display: flex;
  flex-direction: row;
  padding: 16px;
}

.users .list li a {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  align-items: center;
  column-gap: 16px;
  row-gap: 16px;
}

.users .list li p {
  color: var(--white);
}

.users .list li p:hover {
  text-decoration: underline;
}

.users .top input,
.users .list input {
  width: 18px;
  height: 18px;
}

.users .top {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 16px;
  padding: 4px 16px;
}

.users .top p:nth-child(2),
.users .list li a p:nth-child(2) {
  width: 150px;
}

.users .top p:nth-child(3),
.users .list li a p:nth-child(3) {
  width: 190px;
}

.users .top p:nth-child(4),
.users .list li a p:nth-child(4) {
  width: 280px;
}

.users .top p:nth-child(5),
.users .list li a p:nth-child(5) {
  width: 100px;
}

.empty-state {
  text-align: center;
  padding: 20px;
  font-size: 18px;
  color: var(--white);
}
</style>
