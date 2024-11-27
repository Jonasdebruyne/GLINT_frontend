<script setup>
import { ref, reactive, onMounted } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import Navigation from "../../components/navComponent.vue";

// Router instantie en gebruikersvalidatie
const router = useRouter();
const token = localStorage.getItem("jwtToken");

// Als er geen token is, wordt de gebruiker doorgestuurd naar de login pagina
if (!token) {
  router.push("/login");
}

// JWT token parseren
const parseJwt = (token) => {
  try {
    const base64Url = token.split(".")[1];
    const base64 = base64Url.replace(/-/g, "+").replace(/_/g, "/");
    const jsonPayload = atob(base64);
    return JSON.parse(decodeURIComponent(jsonPayload));
  } catch (error) {
    console.error("Fout bij het parsen van JWT:", error);
    return null;
  }
};

const tokenPayload = parseJwt(token);
const userId = tokenPayload?.userId || null;
if (!userId) {
  router.push("/login");
}

// API basis URL configuratie
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
    console.error("Fout bij het ophalen van het gebruikersprofiel:", error);
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
    console.error("Fout bij het ophalen van huisstijl data:", error);
  }
};

// Functie om font te uploaden naar Cloudinary
const uploadFontToCloudinary = async (file) => {
  const formData = new FormData();
  formData.append("file", file);
  formData.append("upload_preset", "ycy4zvmj");
  formData.append("cloud_name", "dzempjvto");
  formData.append("folder", "Stijn/Huisstijl/fonts");

  try {
    const { data } = await axios.post(
      "https://api.cloudinary.com/v1_1/dzempjvto/raw/upload",
      formData
    );
    return data.secure_url;
  } catch (error) {
    console.error("Fout bij het uploaden van het font:", error);
    throw error;
  }
};

// Functie om font-bestand te verwerken
const handleFontFileChange = async (event) => {
  const files = event.target.files;
  if (files?.length > 0) {
    const file = files[0];
    try {
      // Upload het fontbestand naar Cloudinary
      const uploadedFontUrl = await uploadFontToCloudinary(file);

      // Voeg het font toe aan de lijst van huisstijl fonts
      huisstijlData.fonts.push({ url: uploadedFontUrl, name: file.name });

      // Sla de gewijzigde huisstijl data op
      await updateHuisstijlDataJson();
    } catch (error) {
      console.error("Fout bij het verwerken van het fontbestand:", error);
    }
  }
};

// Functie om huisstijl data naar Cloudinary bij te werken
const updateHuisstijlDataJson = async () => {
  try {
    // Haal de huidige huisstijl data op van Cloudinary
    const response = await axios.get(
      "https://res.cloudinary.com/dzempjvto/raw/upload/v1732653110/Stijn/Huisstijl/huisstijl_data.json"
    );
    const currentData = response.data;

    // Werk de gekleurde velden bij
    currentData.primaryColor = huisstijlData.primaryColor;
    currentData.secondaryColor = huisstijlData.secondaryColor;
    currentData.titleColor = huisstijlData.titleColor;
    currentData.colorForButtons = huisstijlData.colorForButtons;
    currentData.fonts = huisstijlData.fonts;

    // Upload de gewijzigde JSON naar Cloudinary
    const formData = new FormData();
    formData.append(
      "file",
      new Blob([JSON.stringify(currentData)], { type: "application/json" })
    );
    formData.append("upload_preset", "ycy4zvmj");
    formData.append("resource_type", "raw");
    formData.append("public_id", "huisstijl_data");
    formData.append("folder", "Stijn/Huisstijl");

    // Upload de JSON naar Cloudinary
    await axios.post(
      "https://api.cloudinary.com/v1_1/dzempjvto/raw/upload",
      formData
    );

    // Werk huisstijlData bij met de nieuwste gegevens
    Object.assign(huisstijlData, currentData);

    // Sla de bijgewerkte huisstijlData lokaal op
    localStorage.setItem("huisstijlData", JSON.stringify(huisstijlData));
  } catch (error) {
    console.error("Fout bij het updaten van huisstijl_data.json:", error);
  }
};

// Refs voor bestandsinvoer
const fontInput = ref(null);

// Functie om de bestandinvoer voor fonts aan te roepen
const triggerFontInput = () => {
  fontInput.value?.click();
};

fetchHuisstijlData();
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
              @click="openColorPicker('primaryColor')"
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
              @click="openColorPicker('secondaryColor')"
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
              @click="openColorPicker('titleColor')"
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
              @click="openColorPicker('colorForButtons')"
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
          <div class="row">
            <p
              :style="{
                fontFamily: font.name ? '${font.name}' : 'sans-serif',
              }"
            >
              AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz​
            </p>
            <input
              type="file"
              ref="fontInput"
              @change="handleFileChange($event, 'logo')"
              style="display: none"
            />
            <button @click="triggerFileInput('logo')">Upload</button>
          </div>
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
            <button @click="triggerFileInput('logo')">Upload</button>
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
            <button @click="triggerFileInput('backgroundImage')">Upload</button>
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

.elements .colours {
  display: flex;
  flex-direction: column;
  gap: 16px;
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

button {
  background-color: var(--purple);
  color: var(--white);
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
