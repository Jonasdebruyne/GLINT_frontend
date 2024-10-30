<script lang="ts" setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import Navigation from "../../components/navComponent.vue";

const jwtToken = localStorage.getItem("jwtToken");
const router = useRouter();

if (!jwtToken) {
  router.push("/login");
}

const firstname = ref<string>("");
const lastname = ref<string>("");
const email = ref<string>("");
const password = ref<string>(""); // Nieuw wachtwoord veld
const role = ref<string>("user");
const status = ref<string>("active");

const isValidEmail = (email: string) => {
  const re = /\S+@\S+\.\S+/;
  return re.test(email);
};

const addUser = async () => {
  if (
    !firstname.value ||
    !lastname.value ||
    !email.value ||
    !password.value ||
    !role.value ||
    !status.value
  ) {
    alert("Vul alle velden in.");
    return;
  }

  if (!isValidEmail(email.value)) {
    alert("Vul een geldig e-mailadres in.");
    return;
  }

  try {
    const userResponse = await fetch(
      "http://localhost:3000/api/v1/users/signup",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({
          user: {
            firstname: firstname.value,
            lastname: lastname.value,
            email: email.value,
            password: password.value,
            role: role.value,
            activeUnactive: status.value,
          },
        }),
      }
    );

    if (!userResponse.ok) {
      const errorDetail = await userResponse.text(); // Of userResponse.json() als de response in JSON is
      throw new Error(
        `HTTP error! status: ${userResponse.status}, details: ${errorDetail}`
      );
    }

    const userResult = await userResponse.json();
    const userId = userResult.data.user.id; // Zorg ervoor dat deze structuur klopt
    const houseStyle = {
      primary_color: "#0071e3",
      secondary_color: "#9747ff",
      background_color: "#e5e5e5",
      text_color: "#000000",
      fontFamilyBodyText: "DM Sans",
      fontFamilyTitles: "Syne",
      logo_url: "assets/images/logo.png",
      userId: userId,
    };

    const houseStyleResponse = await fetch(
      "http://localhost:3000/api/v1/housestyle",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify({ houseStyle }), // Gebruik de correcte casing
      }
    );

    if (!houseStyleResponse.ok) {
      const errorDetail = await houseStyleResponse.text();
      throw new Error(
        `HTTP error! status: ${houseStyleResponse.status}, details: ${errorDetail}`
      );
    }

    const houseStyleResult = await houseStyleResponse.json();

    router.push("./users");
  } catch (error) {
    console.error("Error adding user or creating house style:", error);
    alert(
      "Er is een fout opgetreden bij het toevoegen van de gebruiker of het aanmaken van de huisstijl."
    );
  }
};
</script>

<template>
  <Navigation />
  <div class="content">
    <h1>Add New User</h1>
    <form @submit.prevent="addUser">
      <div class="row">
        <div class="column">
          <label for="firstname">First Name:</label>
          <input v-model="firstname" id="firstname" type="text" required />
        </div>
        <div class="column">
          <label for="lastname">Last Name:</label>
          <input v-model="lastname" id="lastname" type="text" required />
        </div>
      </div>
      <div class="row">
        <div class="column">
          <label for="email">Email:</label>
          <input v-model="email" id="email" type="email" required />
        </div>

        <div class="column">
          <label for="password">Password:</label>
          <input v-model="password" id="password" type="password" required />
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="role">Role:</label>
          <select v-model="role" id="role">
            <option value="user">User</option>
            <option value="admin">Admin</option>
          </select>
        </div>

        <div class="column">
          <label for="status">Status:</label>
          <select v-model="status" id="status">
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
          </select>
        </div>
      </div>

      <button type="submit" class="btn active">Add User</button>
    </form>
  </div>
</template>

<style scoped>
.content {
  width: 100%;
  height: 100vh;
}

form {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 16px;
  width: 100%;
}

form .row {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  justify-content: space-between;
  gap: 120px;
  width: 100%;
}

form .column {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
}

input,
select {
  padding: 8px;
  margin-bottom: 16px;
  border-radius: 4px;
  border: 1px solid #333;
  background-color: #333;
  color: white;
}

button {
  color: white;
  cursor: pointer;
}
</style>
