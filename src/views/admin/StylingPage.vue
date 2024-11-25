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
const searchTerm = ref("");
const selectedFilter = ref("All");
const selectedUsers = ref([]); // Dit is een ref voor de geselecteerde gebruikers
const isPopupVisible = ref(false);

// Ophalen van gebruikersgegevens

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

const fetchData = async () => {
  try {
    const token = localStorage.getItem("jwtToken");

    if (!houseStyleId.value) {
      console.error("No houseStyleId available for API call.");
      return;
    }

    const apiUrl = `${baseURL}/housestyle/${houseStyleId.value}`;

    const response = await axios.get(apiUrl, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });

    const result = response.data;

    const houseStyle = result?.data?.houseStyles?.[0];
    if (houseStyle) {
      backgroundColor.value = houseStyle.background_color;
      bodyTextColor.value = houseStyle.text_color;
      titleColor.value = houseStyle.primary_color;
      buttonColor.value = houseStyle.secondary_color;
      fontFamilyBodyText.value = houseStyle.fontFamilyBodyText;
      fontFamilyTitles.value = houseStyle.fontFamilyTitles;
      logoUrl.value = houseStyle.logo_url;
    } else {
      console.warn("House style data is empty or malformed.");
    }
  } catch (error) {
    console.error("Error fetching data:", error);
  }
};

const saveColor = async () => {
  try {
    const token = localStorage.getItem("jwtToken");

    const colorData = {
      houseStyle: {
        primary_color: titleColor.value,
        secondary_color: buttonColor.value,
        background_color: backgroundColor.value,
        text_color: bodyTextColor.value,
        fontFamilyBodyText: "DM Sans",
        fontFamilyTitles: "Syne",
        logo_url: "assets/images/logo.png",
      },
    };

    const apiUrl = `${baseURL}/housestyle/${userId.value || userId}`;

    const response = await axios.put(apiUrl, colorData, {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
  } catch (error) {
    console.error("Fout bij het opslaan van de kleur:", error.response.data);
  }
};

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

const userRole = ref(null);
const houseStyleId = ref(null);
const backgroundColor = ref("#ffffff");
const bodyTextColor = ref("#000000");
const titleColor = ref("#000000");
const textColor = ref("#000000");
const buttonColor = ref("#000000");
const primaryColor = ref("#000000");
const secondaryColor = ref("#000000");
const fontFamilyBodyText = ref("DM Sans");
const fontFamilyTitles = ref("Syne");
const logoUrl = ref("");

async function changeColor(colorType) {
  const colorPicker = document.createElement("input");
  colorPicker.type = "color";

  colorPicker.value = colorType.value;

  colorPicker.addEventListener("input", async (event) => {
    const newColor = event.target.value;

    if (colorType === titleColor) {
      titleColor.value = newColor;
    } else if (colorType === buttonColor) {
      buttonColor.value = newColor;
    } else if (colorType === backgroundColor) {
      backgroundColor.value = newColor;
    } else if (colorType === bodyTextColor) {
      bodyTextColor.value = newColor;
    }

    await saveColor();
  });

  colorPicker.click();
}

function hexToRgb(hex) {
  hex = hex.replace(/^#/, "");
  let r = parseInt(hex.substring(0, 2), 16);
  let g = parseInt(hex.substring(2, 4), 16);
  let b = parseInt(hex.substring(4, 6), 16);
  return `rgb(${r}, ${g}, ${b})`;
}
</script>

<template>
  <Navigation />
  <div class="content">
    <h1>Styling</h1>
    <div class="elements">
      <div class="top">
        <h2>Colours</h2>
        <a href="#" class="btn" @click.prevent="resetHouseStyle">Reset</a>
      </div>
      <div class="colours">
        <div class="column">
          <h3>Background color</h3>
          <div class="row">
            <p>{{ backgroundColor }} - {{ hexToRgb(backgroundColor) }}</p>
            <div
              class="color"
              :style="{ backgroundColor: backgroundColor }"
              @click="() => changeColor(backgroundColor)"
            ></div>
          </div>
        </div>

        <div class="column">
          <h3>Body text color</h3>
          <div class="row">
            <p>{{ bodyTextColor }} - {{ hexToRgb(bodyTextColor) }}</p>
            <div
              class="color"
              :style="{ color: bodyTextColor }"
              @click="() => changeColor(bodyTextColor)"
            ></div>
          </div>
        </div>
        <div class="column">
          <h3>Title color</h3>
          <div class="row">
            <p>{{ titleColor }} - {{ hexToRgb(titleColor) }}</p>

            <div
              class="color"
              :style="{ color: titleColor }"
              @click="() => changeColor(titleColor)"
            ></div>
          </div>
        </div>
        <div class="column">
          <h3>Color for buttons</h3>
          <div class="row">
            <p>{{ buttonColor }} - {{ hexToRgb(buttonColor) }}</p>
            <div
              class="color"
              :style="{ backgroundColor: buttonColor }"
              @click="() => changeColor(buttonColor)"
            ></div>
          </div>
        </div>
      </div>

      <div class="fonts">
        <h2>Fonts</h2>
        <div class="column">
          <h3>Body text</h3>
          <div class="row">
            <div class="column">
              <label>Name</label>
              <p>{{ fontFamilyBodyText }}</p>
            </div>
            <div class="column">
              <label>Preview</label>
              <p :style="{ fontFamily: fontFamilyBodyText }">
                AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz​
              </p>
            </div>
          </div>
        </div>
        <div class="column">
          <h3>Titles</h3>
          <div class="row">
            <div class="column">
              <label>Name</label>
              <p>{{ fontFamilyTitles }}</p>
            </div>
            <div class="column">
              <label>Preview</label>
              <p :style="{ fontFamily: fontFamilyTitles }">
                AaBbCcDdEeFfGgHhIiJjKkLlMmNnOoPpQqRrSsTtUuVvWwXxYyZz​
              </p>
            </div>
          </div>
        </div>
      </div>
      <div class="images">
        <h2>Images</h2>
        <div class="row">
          <h3>Logo</h3>
          <div class="column">
            <p>{{ logoUrl }}</p>
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

.column .row .color {
  width: 24px;
  height: 24px;
  border: 3px solid var(--gray);
  border-radius: 50%;
  cursor: pointer;
}
</style>
