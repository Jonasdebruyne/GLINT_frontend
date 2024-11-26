<script setup>
import { ref, reactive, onMounted } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import Navigation from "../../components/navComponent.vue";

// Router instance
const router = useRouter();

// JWT token en gebruikersvalidatie
const token = localStorage.getItem("jwtToken");
if (!token) {
  router.push("/login");
}

const parseJwt = (token) => {
  try {
    const base64Url = token.split(".")[1];
    const base64 = base64Url.replace(/-/g, "+").replace(/_/g, "/");
    const jsonPayload = atob(base64);
    return JSON.parse(decodeURIComponent(jsonPayload));
  } catch (error) {
    console.error("Error parsing JWT:", error);
    return null;
  }
};

const tokenPayload = parseJwt(token);
const userId = tokenPayload?.userId || null;
if (!userId) {
  router.push("/login");
}

// API basis URL configureren
const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

// Reactieve data voor het gebruikersprofiel en huisstijl
const user = ref({
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

const huisstijlData = reactive({
  primaryColor: "#9747ff",
  secondaryColor: "#000000",
  titleColor: "#ffffff",
  colorForButtons: "#0071e3",
  fonts: [],
  logo: "",
  backgroundImage: "",
});

// Functie om het gebruikersprofiel op te halen
const fetchUserProfile = async () => {
  try {
    const { data } = await axios.get(`${baseURL}/users/${userId}`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    Object.assign(user, data?.data?.user);
  } catch (error) {
    console.error("Error fetching user profile:", error);
  }
};

// Functie om huisstijl data op te halen
const fetchHuisstijlData = async () => {
  const cachedHuisstijlData = localStorage.getItem("huisstijlData");
  if (cachedHuisstijlData) {
    Object.assign(huisstijlData, JSON.parse(cachedHuisstijlData));
    console.log("Huisstijl data geladen uit cache:", huisstijlData);
    return;
  }

  try {
    const response = await axios.get(
      `https://res.cloudinary.com/dzempjvto/raw/upload/v1732653110/Stijn/Huisstijl/huisstijl_data.json?nocache=${new Date().getTime()}`
    );
    const data = response.data;
    localStorage.setItem("huisstijlData", JSON.stringify(data));
    Object.assign(huisstijlData, data);
    console.log("Huisstijl data geladen:", data);
  } catch (error) {
    console.error("Error fetching huisstijl data:", error);
  }
};

// Functie om huisstijl data naar Cloudinary bij te werken
const updateHuisstijlDataJson = async (newImageUrl, inputName) => {
  try {
    const response = await axios.get(
      "https://res.cloudinary.com/dzempjvto/raw/upload/v1732653110/Stijn/Huisstijl/huisstijl_data.json"
    );

    console.log("Huidige huisstijl data:", response.data); // Log de huidige JSON data

    const currentData = response.data;

    // Update de juiste afbeelding-URL op basis van de inputName
    if (inputName === "logo") {
      currentData.logo = newImageUrl; // Bijwerken van logo-URL
      console.log("Logo URL geüpdatet naar:", newImageUrl);
    } else if (inputName === "backgroundImage") {
      currentData.backgroundImage = newImageUrl; // Bijwerken van background image URL
      console.log("Background image URL geüpdatet naar:", newImageUrl);
    }

    // Maak een nieuwe Blob van de bijgewerkte JSON
    const formData = new FormData();
    formData.append(
      "file",
      new Blob([JSON.stringify(currentData)], { type: "application/json" })
    );
    formData.append("upload_preset", "ycy4zvmj"); // Gebruik je Cloudinary upload preset
    formData.append("resource_type", "raw"); // Gegevens uploaden als raw bestand
    formData.append("public_id", "huisstijl_data"); // Public ID van het bestand
    formData.append("folder", "Stijn/Huisstijl"); // De Cloudinary folder

    // Upload de bijgewerkte JSON naar Cloudinary
    const uploadResponse = await axios.post(
      "https://api.cloudinary.com/v1_1/dzempjvto/raw/upload", // Cloudinary upload endpoint
      formData
    );

    console.log(
      "Geüpdatete huisstijl_data.json succesvol geüpload:",
      uploadResponse.data
    );

    // Haal de nieuwe secure_url van het geüploade bestand
    const updatedJsonUrl = uploadResponse.data.secure_url;
    console.log("Nieuwe URL van huisstijl_data.json:", updatedJsonUrl);

    // Update huisstijlData object met de nieuwe gegevens
    huisstijlData.logo = currentData.logo;
    huisstijlData.backgroundImage = currentData.backgroundImage;
    huisstijlData.primaryColor = currentData.primaryColor;
    huisstijlData.secondaryColor = currentData.secondaryColor;
    huisstijlData.titleColor = currentData.titleColor;
    huisstijlData.colorForButtons = currentData.colorForButtons;
    huisstijlData.fonts = currentData.fonts;

    // Update de cache in localStorage
    localStorage.setItem("huisstijlData", JSON.stringify(huisstijlData));

    console.log("Huisstijl data bijgewerkt:", huisstijlData);
  } catch (error) {
    console.error("Fout bij het updaten van huisstijl_data.json:", error);
  }
};

// Functie om bestand te uploaden naar Cloudinary
const uploadToCloudinary = async (file) => {
  const formData = new FormData();
  formData.append("file", file);
  formData.append("upload_preset", "ycy4zvmj");
  formData.append("cloud_name", "dzempjvto");
  formData.append("folder", "Stijn/Huisstijl");

  try {
    const { data } = await axios.post(
      `https://api.cloudinary.com/v1_1/dzempjvto/image/upload`,
      formData
    );
    return data.secure_url;
  } catch (error) {
    console.error("Error uploading file:", error);
    throw error;
  }
};

// Functie om bestandveranderingen te verwerken
const handleFileChange = async (event, inputName) => {
  const files = event.target.files;
  if (files?.length > 0) {
    const file = files[0];
    console.log("File selected:", file.name);

    try {
      const uploadedUrl = await uploadToCloudinary(file);
      console.log("Uploaded file URL:", uploadedUrl);

      if (inputName === "logo") huisstijlData.logo = uploadedUrl;
      else if (inputName === "backgroundImage")
        huisstijlData.backgroundImage = uploadedUrl;

      await updateHuisstijlDataJson(uploadedUrl, inputName);
    } catch (error) {
      console.error("Error handling file change:", error);
    }
  }
};

// Functie om huisstijl data te resetten
const resetHouseStyle = () => {
  Object.assign(huisstijlData, {
    primaryColor: "#000000",
    secondaryColor: "#FFFFFF",
    titleColor: "#000000",
    colorForButtons: "#007BFF",
    fonts: [],
  });
  console.log("Huisstijl reset naar standaard waarden.");
};

// Refs voor bestandsinvoer
const logoInput = ref(null);
const backgroundInput = ref(null);

// Functie om het bestand invoer aan te roepen
const triggerFileInput = (inputName) => {
  const fileInput =
    inputName === "logo" ? logoInput.value : backgroundInput.value;
  fileInput?.click();
};

fetchHuisstijlData(); // Alleen laden wanneer de component wordt geladen
fetchUserProfile();
</script>

<template>
  <Navigation />
  <div class="content">
    <h1>Styling</h1>
    <div class="elements">
      <!-- Colours Section -->
      <div class="top">
        <h2>Colours</h2>
        <a href="#" class="btn" @click.prevent="resetHouseStyle">Reset</a>
      </div>
      <div class="colours">
        <div class="column">
          <h3>Primary Color</h3>
          <div class="row">
            <p>{{ huisstijlData.primaryColor }}</p>
            <div
              class="color"
              :style="{ backgroundColor: huisstijlData.primaryColor }"
            ></div>
          </div>
        </div>
        <div class="column">
          <h3>Secondary Color</h3>
          <div class="row">
            <p>{{ huisstijlData.secondaryColor }}</p>
            <div
              class="color"
              :style="{ backgroundColor: huisstijlData.secondaryColor }"
            ></div>
          </div>
        </div>
        <div class="column">
          <h3>Title color</h3>
          <div class="row">
            <p>{{ huisstijlData.titleColor }}</p>
            <div
              class="color"
              :style="{ backgroundColor: huisstijlData.titleColor }"
            ></div>
          </div>
        </div>
        <div class="column">
          <h3>Color for buttons</h3>
          <div class="row">
            <p>{{ huisstijlData.colorForButtons }}</p>
            <div
              class="color"
              :style="{ backgroundColor: huisstijlData.colorForButtons }"
            ></div>
          </div>
        </div>
      </div>

      <!-- Fonts Section -->
      <div class="fonts">
        <h2>Fonts</h2>
        <div
          v-for="(font, index) in huisstijlData.fonts"
          :key="index"
          class="column"
        >
          <h3>{{ font.name || "No Font" }}</h3>
          <p
            :style="{ fontFamily: font.name ? `'${font.name}'` : 'sans-serif' }"
          >
            AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz​
          </p>
        </div>
      </div>

      <div class="images">
        <h2>Images</h2>
        <div class="column">
          <h3>Logo</h3>
          <div class="row">
            <img :src="huisstijlData.logo" alt="Logo" />
            <!-- Bestandinvoer voor logo -->
            <input
              type="file"
              ref="logoInput"
              @change="handleFileChange($event, 'logo')"
              style="display: none"
            />
            <button @click="triggerFileInput('logo')">Upload Logo</button>
          </div>
        </div>

        <div class="column">
          <h3>Background Image</h3>
          <div class="row">
            <img :src="huisstijlData.backgroundImage" alt="Background Image" />
            <!-- Bestandinvoer voor achtergrondafbeelding -->
            <input
              type="file"
              ref="backgroundInput"
              @change="handleFileChange($event, 'backgroundImage')"
              style="display: none"
            />
            <button @click="triggerFileInput('backgroundImage')">
              Upload Background Image
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.elements {
  background-color: #1d1d1d;
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 24px;
  padding: 24px;
  border-radius: 8px;
}

.elements .top {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  padding-bottom: 12px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.elements .top h2 {
  padding-bottom: 0;
  border-bottom: none;
}

.elements h2 {
  padding-bottom: 12px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.elements .colours,
.elements .fonts,
.elements .images {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.elements .images img {
  width: 64px;
  height: 64px;
}

.elements .images .row {
  display: flex;
  flex-direction: row;
  align-items: flex-end;
  justify-content: space-between;
  width: 100%;
}

.elements .images .row button {
  background-color: var(--gray);
  border-radius: 8px;
  padding: 4px 12px;
}

.column {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.column .row,
.images .row {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.column .row .column {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.column .row .color {
  width: 24px;
  height: 24px;
  border: 3px solid var(--gray);
  border-radius: 50%;
  cursor: pointer;
}
</style>
