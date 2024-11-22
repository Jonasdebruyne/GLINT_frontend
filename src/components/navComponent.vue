<script setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import axios from "axios"; // Ensure axios is installed

const router = useRouter();
const user = ref({ firstname: "", lastname: "", role: "" }); // Initialize with empty values

const getUserDataFromToken = () => {
  const token = localStorage.getItem("jwtToken");
  if (token) {
    const payload = token.split(".")[1];
    const decodedPayload = JSON.parse(atob(payload));
    const userId = decodedPayload.userId;
    const firstname = decodedPayload.firstname;
    const lastname = decodedPayload.lastname;
    const role = decodedPayload.role;

    if (!userId) {
      console.error("User ID not found in token");
    }

    return { userId, firstname, lastname, role }; // Return userId, firstname en lastname
  }
  console.error("No token found in localStorage");
  return null;
};

// Bij het ophalen van de gebruikersdata in onMounted, pas deze functie aan:
onMounted(() => {
  const userData = getUserDataFromToken();
  if (userData) {
    user.value.firstname = userData.firstname; // Stel firstname in
    user.value.lastname = userData.lastname; // Stel lastname in
    user.value.role = userData.role; // Stel lastname in
    fetchUserData(userData.userId); // Roep de API aan met userId
  }
});

const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

const fetchUserData = async (userId) => {
  try {
    const response = await axios.get(`${baseURL}/users/${userId}`);

    const userData = response.data.data.user;
    if (userData) {
      user.value = {
        firstname: userData.firstname || "",
        lastname: userData.lastname || "",
        role: userData.role || "",
      };
    } else {
      console.error("No user found in the response.");
    }
  } catch (error) {
    console.error("Error fetching user data:", error);
  }
};

const logout = () => {
  localStorage.removeItem("jwtToken");
  router.push("/login");
};
</script>

<template>
  <nav>
    <div class="logo"></div>
    <div class="elements">
      <div class="profile">
        <div class="profilePicture"></div>
        <div>
          <h3 v-if="user.firstname && user.lastname">
            {{ user.firstname }} {{ user.lastname }}
          </h3>
          <p v-if="user.role">
            {{ user.role }}
          </p>
        </div>
      </div>
      <div class="menu">
        <router-link to="/admin" exact-active-class="active">
          <img src="../assets/icons/package.svg" alt="icon" />
          <p>Products</p>
          ,
        </router-link>
        <router-link to="/admin/orders" exact-active-class="active">
          <img src="../assets/icons/order.svg" alt="icon" />
          <p>Orders</p>
        </router-link>
        <router-link
          v-if="user.role === 'partner_owner'"
          to="/admin/styling"
          exact-active-class="active"
        >
          <img src="../assets/icons/paintbrush.svg" alt="icon" />
          <p>Styling</p>
        </router-link>
        <router-link to="/admin/users" exact-active-class="active">
          <img src="../assets/icons/users.svg" alt="icon" />
          <p>Users</p>
        </router-link>
        <router-link to="/admin/settings" exact-active-class="active">
          <img src="../assets/icons/settings.svg" alt="icon" />
          <p>Settings</p>
        </router-link>
        <a @click.prevent="logout">
          <img src="../assets/icons/logout.svg" alt="icon" />
          <p>Logout</p>
        </a>
      </div>
    </div>
  </nav>
</template>

<style scoped>
nav {
  display: flex;
  flex-direction: column;
  gap: 120px;
  background: linear-gradient(to bottom, #000000, #473c5d);
  height: 100vh;
  padding: 48px 32px;
  position: fixed;
}

nav .logo {
  background-image: url("../assets/images/GLINT-logo-white.png");
  background-position: center;
  background-size: contain;
  background-repeat: no-repeat;
  width: 64px;
  height: 24px;
}

nav .elements,
nav .elements .menu {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 64px;
}

nav .elements .profile {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

nav .elements .profile .profilePicture {
  background-image: url("../assets/images/Odette_lunettes.webp");
  background-position: center;
  background-size: cover;
  background-repeat: no-repeat;
  width: 64px;
  height: 64px;
  border-radius: 50%;
}

nav .elements .profile div {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

nav .elements .menu a,
nav .elements .menu a.active {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 8px;
}

nav .elements .menu a.active {
  background-color: var(--purple);
  padding: 4px 12px;
  border-radius: 8px;
}

nav .elements .menu a img {
  width: 24px;
}

nav .elements .menu a p {
  color: var(--white);
}
</style>
