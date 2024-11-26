<script setup>
import { ref, reactive, onMounted, provide } from "vue";
import { useRouter } from "vue-router";
import axios from "axios";
import Navigation from "../../components/navComponent.vue";

// Router instance
const router = useRouter();

// Reactive data for user profile and huisstijl (styling) data
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

provide("user", user);

const huisstijlData = reactive({
  primaryColor: "",
  secondaryColor: "",
  titleColor: "",
  colorForButtons: "",
  fonts: [],
  logo: "",
  backgroundImage: "",
});

// JWT token and user validation
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
const userId = tokenPayload?.userId || null;

if (!userId) {
  router.push("/login");
}

// Base API URL
const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

// Fetch user profile
const fetchUserProfile = async () => {
  try {
    const response = await axios.get(`${baseURL}/users/${userId}`, {
      headers: { Authorization: `Bearer ${token}` },
    });
    const userData = response.data?.data?.user || {};
    Object.assign(user, {
      firstName: userData.firstname || "",
      lastName: userData.lastname || "",
      email: userData.email || "",
      oldEmail: userData.email || "",
      country: userData.country || "",
      city: userData.city || "",
      postalCode: userData.postalCode || "",
      profilePicture: userData.profilePicture || "",
      bio: userData.bio || "",
      role: userData.role || "",
      activeUnactive: userData.activeUnactive ?? true,
    });
  } catch (error) {
    console.error("Error fetching user profile:", error);
  }
};

// Fetch huisstijl (styling) data
const loadFonts = (fonts) => {
  fonts.forEach((font) => {
    const fontFace = new FontFace(font.name, `url(${font.path})`);
    fontFace
      .load()
      .then((loadedFace) => {
        document.fonts.add(loadedFace);
        console.log(`Font ${font.name} loaded successfully.`);
      })
      .catch((error) => {
        console.error(`Failed to load font ${font.name}:`, error);
      });
  });
};

const fetchHuisstijlData = async () => {
  try {
    const response = await axios.get(
      "https://res.cloudinary.com/dzempjvto/raw/upload/v1732653110/Stijn/Huisstijl/huisstijl_data.json"
    );

    console.log("Huisstijl data succesvol opgehaald:", response.data);

    const data = response.data;
    Object.assign(huisstijlData, {
      primaryColor: data.primaryColor || huisstijlData.primaryColor,
      secondaryColor: data.secondaryColor || huisstijlData.secondaryColor,
      titleColor: data.titleColor || huisstijlData.titleColor,
      colorForButtons: data.colorForButtons || huisstijlData.colorForButtons,
      fonts: data.fonts || huisstijlData.fonts,
      logo: data.logo || huisstijlData.logo,
      backgroundImage: data.backgroundImage || huisstijlData.backgroundImage,
    });

    if (huisstijlData.fonts?.length) {
      loadFonts(huisstijlData.fonts);
    }
    console.log("Huisstijl data geladen:", data);
  } catch (error) {
    console.error("Error fetching huisstijl data:", error);
  }
};

// Reset huisstijl data to defaults
const resetHouseStyle = () => {
  Object.assign(huisstijlData, {
    primaryColor: "#000000",
    secondaryColor: "#FFFFFF",
    titleColor: "#000000",
    colorForButtons: "#007BFF",
    fonts: [],
  });
  console.log("Huisstijl reset to default values.");
};

// Refs for file inputs
const logoInput = ref(null);
const backgroundInput = ref(null);

// Function to trigger the file input click
const triggerFileInput = (inputName) => {
  let fileInput;
  if (inputName === "logo") {
    fileInput = logoInput.value;
  } else if (inputName === "backgroundImage") {
    fileInput = backgroundInput.value;
  }

  if (fileInput) {
    fileInput.click();
  } else {
    console.error(`File input with name '${inputName}' not found!`);
  }
};

const updateHuisstijlDataJson = async (newImageUrl, inputName) => {
  try {
    // Fetch the current huisstijl_data.json from Cloudinary
    const response = await axios.get(
      "https://res.cloudinary.com/dzempjvto/raw/upload/v1732575816/Stijn/Huisstijl/huisstijl_data.json"
    );

    console.log("Current huisstijl data:", response.data); // Log the current JSON data

    const currentData = response.data;

    // Update the logo or background image based on inputName
    if (inputName === "logo") {
      currentData.logo = newImageUrl; // Update logo URL
    } else if (inputName === "backgroundImage") {
      currentData.backgroundImage = newImageUrl; // Update background image URL
    }

    console.log("Updated huisstijl data:", currentData); // Log the updated JSON data

    // Convert the updated JSON data into a Blob
    const jsonString = JSON.stringify(currentData);
    const blob = new Blob([jsonString], { type: "application/json" });

    // Prepare FormData for uploading the JSON file to Cloudinary
    const formData = new FormData();
    formData.append("file", blob); // Append the JSON Blob
    formData.append("upload_preset", "ycy4zvmj"); // Use your Cloudinary upload preset
    formData.append("resource_type", "raw"); // Set resource type to "raw"
    formData.append("public_id", "huisstijl_data"); // Public ID for the JSON file (use a different one if necessary)
    formData.append("folder", "Stijn/Huisstijl"); // Specify the folder

    // Upload the JSON file back to Cloudinary
    const uploadResponse = await axios.post(
      "https://api.cloudinary.com/v1_1/dzempjvto/raw/upload", // Cloudinary upload API endpoint
      formData
    );

    console.log(
      "Huisstijl data JSON successfully uploaded:",
      uploadResponse.data
    );
  } catch (error) {
    console.error("Error updating huisstijl_data.json:", error);
  }
};

// Handle file selection and upload to Cloudinary
const uploadToCloudinary = async (file, inputName) => {
  const formData = new FormData();
  formData.append("file", file);
  formData.append("upload_preset", "ycy4zvmj");
  formData.append("cloud_name", "dzempjvto");
  formData.append("folder", "Stijn/Huisstijl");

  try {
    // Log the file to be uploaded
    console.log("Uploading file to Cloudinary:", file.name);

    const response = await axios.post(
      `https://api.cloudinary.com/v1_1/dzempjvto/image/upload`,
      formData
    );

    // Log Cloudinary response
    console.log("Cloudinary upload response:", response);

    if (response?.data?.secure_url) {
      const uploadedImageUrl = response.data.secure_url;
      console.log("Image successfully uploaded. URL:", uploadedImageUrl);

      // Update huisstijl_data.json with the new image URL
      await updateHuisstijlDataJson(uploadedImageUrl, inputName);

      return uploadedImageUrl;
    } else {
      console.error("No secure URL received from Cloudinary.");
      return null;
    }
  } catch (error) {
    console.error("Error uploading file to Cloudinary:", error);
    return null;
  }
};

const initialize = async () => {
  await fetchUserProfile();
  await fetchHuisstijlData();
};

const handleFileChange = (event, inputName) => {
  const files = event.target.files;
  if (files && files.length > 0) {
    const file = files[0];
    if (inputName === "logo") {
      huisstijlData.logo = URL.createObjectURL(file);
    } else if (inputName === "backgroundImage") {
      huisstijlData.backgroundImage = URL.createObjectURL(file);
    }

    // Upload the file to Cloudinary after selection
    uploadToCloudinary(file, inputName);
  } else {
    console.error("No file selected.");
  }
};

// Triggering file upload on mounted
onMounted(() => {
  initialize();
});
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
