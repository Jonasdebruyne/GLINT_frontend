<script lang="ts" setup>
import { ref, onMounted } from "vue";
import Navigation from "../../components/navComponent.vue";
import axios from "axios";
import { useRouter } from "vue-router";

const router = useRouter();

const user = ref({
  firstName: "",
  lastName: "",
  email: "",
  newEmail: "",
  oldEmail: "",
  password: "",
  newpassword: "",
  oldpassword: "",
  newPasswordRepeat: "",
  country: "",
  city: "",
  postalCode: "",
  profilePicture: "",
  bio: "",
  role: "",
  activeUnactive: true,
});

const jwtToken = localStorage.getItem("jwtToken");
if (!jwtToken) {
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

const tokenPayload = parseJwt(jwtToken);
if (!tokenPayload || !tokenPayload.userId) {
  console.error("User ID is not available in the token.");
  router.push("/login");
}

const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

const userId = tokenPayload.userId;
const activeSection = ref("My profile");
const profileEditPopup = ref(false);
const changeEmailAddressPopup = ref(false);
const changePasswordPopup = ref(false);
const deleteAccountPopup = ref(false);

const setActiveSection = (section) => {
  activeSection.value = section;
};

const fetchUserProfile = async () => {
  try {
    const response = await axios.get(`${baseURL}/users/${userId}`, {
      headers: {
        Authorization: `Bearer ${jwtToken}`,
      },
    });

    console.log("User Profile Response:", response.data);

    if (response.data && response.data.data && response.data.data.user) {
      user.value = {
        firstName: response.data.data.user.firstname || null,
        lastName: response.data.data.user.lastname || null,
        email: response.data.data.user.email || null,
        oldEmail: response.data.data.user.email || null,
        country: response.data.data.user.country || null,
        city: response.data.data.user.city || null,
        postalCode: response.data.data.user.postalCode || null,
        profilePicture: response.data.data.user.profilePicture || null,
        bio: response.data.data.user.bio || null,
        role: response.data.data.user.role || null,
        activeUnactive: response.data.data.user.activeUnactive,
      };
    } else {
      console.error("User object is not found in the response data");
    }
  } catch (error) {
    console.error("Error fetching user profile:", error);
  }
};

const updateProfile = async () => {
  try {
    const response = await axios.put(
      `${baseURL}/users/${userId}`,
      {
        user: {
          firstname: user.value.firstName,
          lastname: user.value.lastName,
          email: user.value.email,
          password: user.value.email,
          role: user.value.role,
          activeUnactive: user.value.activeUnactive,
          country: user.value.country,
          city: user.value.city,
          postalCode: user.value.postalCode,
          profileImage: user.value.profilePicture,
          bio: user.value.bio,
        },
      },
      {
        headers: {
          Authorization: `Bearer ${jwtToken}`,
        },
      }
    );

    console.log("Profile updated successfully:", response.data);
    closeProfileEditPopup();
    await fetchUserProfile();
  } catch (error) {
    console.error("Error updating profile:", error);
    alert("There was an error updating your profile. Please try again.");
  }
};

const updateEmailAddress = async () => {
  try {
    const response = await axios.put(
      `${baseURL}/users/${userId}`,
      {
        user: {
          firstname: user.value.firstName,
          lastname: user.value.lastName,
          email: user.value.newEmail,
          password: user.value.email,
          role: user.value.role,
          activeUnactive: user.value.activeUnactive,
          country: user.value.country,
          city: user.value.city,
          postalCode: user.value.postalCode,
          profileImage: user.value.profilePicture,
          bio: user.value.bio,
        },
      },
      {
        headers: {
          Authorization: `Bearer ${jwtToken}`,
        },
      }
    );

    console.log("Profile updated successfully:", response.data);
    closeChangeEmailAddressPopup();
    await fetchUserProfile();
  } catch (error) {
    console.error("Error updating profile:", error);
    alert("There was an error updating your profile. Please try again.");
  }
};

async function updatePassword() {
  try {
    if (
      !user.value.oldpassword ||
      !user.value.newpassword ||
      !user.value.newPasswordRepeat
    ) {
      alert(
        "Vul alstublieft het oude wachtwoord, het nieuwe wachtwoord en de herhaling van het nieuwe wachtwoord in."
      );
      return;
    }

    if (user.value.newpassword !== user.value.newPasswordRepeat) {
      alert(
        "Het nieuwe wachtwoord en de herhaling van het nieuwe wachtwoord komen niet overeen."
      );
      return;
    }

    const response = await axios.put(
      `${baseURL}/users/${userId}`,
      {
        user: {
          oldPassword: user.value.oldpassword, // Oude wachtwoord
          newPassword: user.value.newpassword, // Nieuwe wachtwoord
        },
      },
      {
        headers: {
          Authorization: `Bearer ${jwtToken}`,
        },
      }
    );

    console.log("Old Password:", user.value.oldpassword);
    console.log("New Password:", user.value.newpassword);
    console.log("Password updated successfully:", response.data);
    closeChangePasswordPopup();
  } catch (error) {
    console.error("Error updating password:", error);
    if (error.response && error.response.data && error.response.data.message) {
      alert(error.response.data.message); // Toon het foutbericht van de backend
    } else {
      alert(
        "Er is een fout opgetreden bij het bijwerken van uw wachtwoord. Probeer het opnieuw."
      );
    }
  }
}

const handleDeleteAccount = async () => {
  try {
    const response = await axios.delete(`${baseURL}/users/${userId}`);
    // Hier kun je een melding geven dat de delete succesvol is of de gebruiker navigeren
    console.log("Account deleted successfully", response.data);
    // Bijv. navigeren naar een andere pagina
  } catch (error) {
    console.error(
      "Er ging iets mis bij het verwijderen van het account",
      error
    );
    // Hier kun je een foutmelding tonen aan de gebruiker
  } finally {
    // Sluit de popup, ongeacht of de delete succesvol was of niet
    localStorage.removeItem("jwtToken");
    router.push("/login");
  }
};

const openEditPopup = () => {
  profileEditPopup.value = true;
};

const openChangeEmailAddressPopup = () => {
  changeEmailAddressPopup.value = true;
};

const openChangePasswordPopup = () => {
  changePasswordPopup.value = true;
};

const openDeleteAccountPopup = () => {
  deleteAccountPopup.value = true;
};

const closeProfileEditPopup = () => {
  profileEditPopup.value = false;
};

const closeChangeEmailAddressPopup = () => {
  changeEmailAddressPopup.value = false;
};

const closeChangePasswordPopup = () => {
  changePasswordPopup.value = false;
};

const closeDeleteAccountPopup = () => {
  deleteAccountPopup.value = false;
};

onMounted(() => {
  fetchUserProfile();
});
</script>

<template>
  <Navigation />
  <div class="content">
    <h1>Settings</h1>
    <div class="elements">
      <div class="menu">
        <p
          :class="{ active: activeSection === 'My profile' }"
          @click="setActiveSection('My profile')"
        >
          My profile
        </p>
        <p
          :class="{ active: activeSection === 'Login settings' }"
          @click="setActiveSection('Login settings')"
        >
          Login settings
        </p>
        <p
          :class="{ active: activeSection === 'Field names' }"
          @click="setActiveSection('Field names')"
        >
          Field names
        </p>
      </div>

      <div v-if="activeSection === 'My profile'" class="myProfile">
        <div class="row">
          <h2 class="border">My profile</h2>
          <button @click="openEditPopup" class="btn">
            <img src="../../assets/icons/paintbrush.svg" alt="icon" />
            <p>Edit</p>
          </button>
        </div>
        <div class="info">
          <div class="pictureWithText">
            <div class="profilePicture"></div>
            <div>
              <h3>{{ user.firstName }} {{ user.lastName }}</h3>
              <p>{{ user.bio ? user.bio : user.role }}</p>
            </div>
          </div>
        </div>

        <div class="personalInformation">
          <h3>Personal Information</h3>
          <div class="fields">
            <div class="column">
              <label>First name</label>
              <p>{{ user.firstName }}</p>
            </div>
            <div class="column">
              <label>Last name</label>
              <p>{{ user.lastName }}</p>
            </div>
            <div class="column">
              <label>Email address</label>
              <p>{{ user.email }}</p>
            </div>
          </div>
        </div>

        <div class="address">
          <h3>Address</h3>
          <div class="fields">
            <div class="column">
              <label>Country</label>
              <p>{{ user.country }}</p>
            </div>
            <div class="column">
              <label>City</label>
              <p>{{ user.city }}</p>
            </div>
            <div class="column">
              <label>Postal code</label>
              <p>{{ user.postalCode }}</p>
            </div>
          </div>
        </div>
      </div>

      <div v-if="activeSection === 'Login settings'" class="account">
        <h2 class="border">Account</h2>
        <div class="accountElements">
          <div class="column">
            <h3>Change email</h3>
            <div class="row">
              <p>{{ user.email }}</p>
              <button @click="openChangeEmailAddressPopup" class="link">
                <p>Change email</p>
              </button>
            </div>
          </div>
          <div class="column">
            <h3>Change password</h3>
            <div class="row">
              <p>&bull;&bull;&bull;&bull;&bull;&bull;&bull;&bull;</p>
              <button @click="openChangePasswordPopup" class="link">
                <p>Change password</p>
              </button>
            </div>
          </div>
          <div class="column">
            <div class="row">
              <h3>Delete account</h3>
              <button @click="openDeleteAccountPopup" class="deletelink">
                <p>Delete account</p>
              </button>
            </div>
          </div>
        </div>
      </div>

      <div v-if="activeSection === 'Field names'" class="fieldNames">
        <h2 class="border">Field names</h2>
        <p>Dit zijn de velden van de database.</p>
      </div>
    </div>

    <!-- Popup voor het bewerken van het profiel -->
    <transition name="fade">
      <div v-if="profileEditPopup" class="popup">
        <div class="popup-content">
          <div class="row">
            <span class="close" @click="closeProfileEditPopup">&times;</span>
            <h2>Edit Profile</h2>
          </div>
          <div class="fields">
            <div class="row">
              <div class="column">
                <label>First name</label>
                <input v-model="user.firstName" placeholder="Voornaam" />
              </div>
              <div class="column">
                <label>Last name</label>
                <input v-model="user.lastName" placeholder="Achternaam" />
              </div>
            </div>
            <div class="row">
              <div class="column">
                <label>Email address</label>
                <input v-model="user.email" placeholder="E-mail" />
              </div>
              <div class="column">
                <label>Country</label>
                <input v-model="user.country" placeholder="Land" />
              </div>
            </div>
            <div class="row">
              <div class="column">
                <label>City</label>
                <input v-model="user.city" placeholder="Stad" />
              </div>
              <div class="column">
                <label>Postal code</label>
                <input v-model="user.postalCode" placeholder="Postcode" />
              </div>
            </div>
            <div class="column">
              <label>Profile Picture URL</label>
              <input
                v-model="user.profilePicture"
                placeholder="URL voor profielfoto"
              />
            </div>
            <div class="column">
              <label>Bio</label>
              <textarea v-model="user.bio" placeholder="Bio"></textarea>
            </div>
            <button @click="updateProfile" class="btn">Save</button>
          </div>
        </div>
      </div>
    </transition>
    <transition name="fade">
      <div v-if="changeEmailAddressPopup" class="popup">
        <div class="popup-content">
          <div class="row">
            <span class="close" @click="closeChangeEmailAddressPopup"
              >&times;</span
            >
            <h2>Change Email Address</h2>
          </div>
          <div class="fields">
            <div class="column">
              <label>Old Email Address</label>
              <p>{{ user.email }}</p>
            </div>
            <div class="column">
              <label>New Email Address</label>
              <input v-model="user.newEmail" placeholder="example@mail.be" />
            </div>
            <button @click="updateEmailAddress" class="btn">Save</button>
          </div>
        </div>
      </div>
    </transition>
    <transition name="fade">
      <div v-if="changePasswordPopup" class="popup">
        <div class="popup-content">
          <div class="row">
            <span class="close" @click="closeChangePasswordPopup">&times;</span>
            <h2>Change Password</h2>
          </div>
          <div class="fields">
            <div class="column">
              <label>Old password</label>
              <input
                v-model="user.oldpassword"
                type="password"
                placeholder="••••••••"
              />
            </div>
            <div class="column">
              <label>New password</label>
              <input
                v-model="user.newpassword"
                type="password"
                placeholder="••••••••"
              />
            </div>
            <div class="column">
              <label>Repeat new password</label>
              <input
                type="password"
                v-model="user.newPasswordRepeat"
                placeholder="••••••••"
              />
            </div>
          </div>
          <button
            class="btn"
            @click="updatePassword(user.oldpassword, user.newpassword)"
          >
            Save
          </button>
        </div>
      </div>
    </transition>

    <transition name="fade">
      <div v-if="deleteAccountPopup" class="deletePopup">
        <img
          src="../../assets/icons/cross-circle.svg"
          alt="icon"
          @click="closeDeleteAccountPopup"
        />
        <div class="text">
          <h2>Are you sure?</h2>
          <p>Do you really want to delete your account?</p>
          <div class="btns">
            <button @click="closeDeleteAccountPopup">Cancel</button>
            <button @click="handleDeleteAccount" class="btn active">
              Delete
            </button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
.elements {
  background-color: #1d1d1d;
  width: 100%;
  display: flex;
  flex-direction: row;
  gap: 24px;
  padding: 24px;
  border-radius: 8px;
}

.elements .menu {
  padding-right: 24px;
  border-right: 1px solid rgba(255, 255, 255, 0.2);
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 200px;
}

.elements .menu p {
  padding-left: 24px;
}

.elements .menu p.active {
  background-color: var(--purple);
  padding: 4px 12px 4px 24px;
  border-radius: 8px;
  color: var(--white);
}

.elements .myProfile {
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 100%;
}

.elements .account {
  width: 100%;
}

.elements .myProfile .info {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  justify-content: space-between;
}

.elements .myProfile .info .pictureWithText {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 16px;
}

.elements .myProfile .info .pictureWithText .profilePicture {
  background-image: url("../../assets/images/Odette_lunettes.webp");
  background-position: center;
  background-size: cover;
  background-repeat: no-repeat;
  width: 64px;
  height: 64px;
  border-radius: 50%;
}

.elements .myProfile .info .pictureWithText div {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.info,
.personalInformation,
.address {
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 16px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 8px;
}

.personalInformation,
.address {
  align-items: flex-start;
}

.myProfile .row {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  padding-bottom: 16px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.fieldNames,
.account {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.account .accountElements {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.account .accountElements .column {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.account .accountElements .column .row {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.account .accountElements .column .row .link {
  background-color: var(--gray);
  border-radius: 8px;
  padding: 4px 12px;
}

.account .accountElements .column .row .link p {
  color: var(--black);
  font-weight: 400;
}

.account .accountElements .column .row .deletelink {
  border: 1px solid #d34848;
  background-color: rgba(211, 72, 72, 0.2);
  border-radius: 8px;
  padding: 4px 12px;
}

.account .accountElements .column .row .deletelink p {
  color: #d34848;
}

.fields,
.fieldNames .list {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  column-gap: 120px;
  row-gap: 16px;
}

.fieldNames .list {
  padding: 16px 0;
}

.fields .column {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.column p,
input,
textarea {
  color: rgba(255, 255, 255, 0.4);
  font-weight: 700;
}

input,
textarea {
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 8px;
  padding: 4px 8px;
}

textarea {
  background-color: transparent;
  padding: 8px;
  height: 100px;
}

input,
textarea {
  width: 100%;
}

.elements .display {
  display: none;
}

.popup {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.popup-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  width: 400px;
  position: relative;
  align-items: flex-end;
}

.close {
  position: absolute;
  top: 10px;
  right: 15px;
  cursor: pointer;
}

.popup {
  position: fixed;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.popup-content {
  background-color: #1d1d1d;
  padding: 24px;
  border-radius: 8px;
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.popup-content .row {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 16px;
  width: 100%;
}

.popup-content .fields {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 16px;
  width: 100%;
}

.popup-content .row span {
  font-size: 24px;
}

.popup-content .column {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
}

.popup-content .btns {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.popup-content .btns .btn {
  color: var(--white);
}

button {
  background-color: transparent;
}

.deletePopup {
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

.deletePopup img {
  width: 64px;
  height: 64px;
}

.deletePopup .text {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.deletePopup .text .btns {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.deletePopup .text .btns button {
  color: rgba(255, 255, 255, 0.6);
  border: none;
  background: transparent;
  cursor: pointer;
}

.deletePopup .text .btns .active {
  background-color: #d34848;
  color: var(--white);
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
</style>
