<script lang="ts" setup>
import { ref, onMounted } from "vue";
import { useRouter, useRoute } from "vue-router";
import axios from "axios";

const email = ref("");
const verificationCode = ref("");
const errorMessage = ref("");
const router = useRouter();
const route = useRoute();

const isValidEmail = (email: string) => {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return re.test(email);
};

onMounted(() => {
  if (route.query.email) {
    email.value = route.query.email as string;
  }
});

const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

const sendMail = async () => {
  if (!isValidEmail(email.value)) {
    errorMessage.value = "Voer een geldig e-mailadres in.";
    return;
  }

  try {
    const response = await axios.post(`${baseURL}/users/forgot-password`, {
      email: email.value,
    });

    if (response && response.status >= 200 && response.status < 300) {
      errorMessage.value = "";
      router.push({ path: "/verificationCode", query: { email: email.value } });
    } else {
      errorMessage.value = "Er is een onverwachte fout opgetreden.";
    }
  } catch (error) {
    if (error.response && error.response.status === 404) {
      errorMessage.value = "Gebruiker niet gevonden.";
    } else {
      errorMessage.value =
        "Er is een fout opgetreden bij het versturen van de e-mail.";
    }
  }
};

const verifyCode = async () => {
  if (!verificationCode.value) {
    errorMessage.value = "Voer de verificatiecode in.";
    return;
  }

  try {
    const response = await axios.post(`${baseURL}/users/verify-code`, {
      code: verificationCode.value,
      email: email.value,
    });

    if (response && response.status >= 200 && response.status < 300) {
      errorMessage.value = "";
      router.push("/reset-password");
    } else {
      errorMessage.value = "Er is een onverwachte fout opgetreden.";
    }
  } catch (error) {
    if (error.response) {
      errorMessage.value =
        "Er is een fout opgetreden bij het verifiëren van de code.";
    } else {
      errorMessage.value = "Netwerkfout.";
    }
  }
};
</script>

<template>
  <div class="container">
    <div class="overlay">
      <div class="elements">
        <h1>Forgot password</h1>
        <form @submit.prevent="sendMail">
          <div class="alignItems">
            <div class="column">
              <label for="email">Email</label>
              <input
                id="email"
                v-model="email"
                type="email"
                placeholder="johndoe@gmail.com"
                required
              />
            </div>
            <router-link exact to="./login">Back to login</router-link>
          </div>
          <p v-if="errorMessage" class="error">{{ errorMessage }}</p>

          <button class="submitBtn" type="submit">Send mail</button>
        </form>

        <form v-if="verificationCode" @submit.prevent="verifyCode">
          <div class="column">
            <label for="verificationCode">Verification code</label>
            <input
              id="verificationCode"
              v-model="verificationCode"
              type="text"
              placeholder="HFDQ2FDQL4"
              required
            />
          </div>

          <p v-if="errorMessage" class="error">{{ errorMessage }}</p>

          <button class="submitBtn" type="submit">Verify code</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Je bestaande CSS-stijlen hier */
.container {
  position: relative;
  background-image: url("../assets/images/background.jpg");
  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
  width: 100%;
  height: 100vh;
}

.overlay {
  background-color: rgba(0, 0, 0, 0.6);
  width: 100%;
  height: 100vh;
}

.elements {
  position: absolute;
  top: 50%;
  right: 0;
  transform: translateY(-50%);
  width: 50%;
  background-color: rgba(0, 0, 0, 0.32);
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 48px;
  padding: 48px;
  border-radius: 8px;
  margin: 48px;
}

form {
  display: flex;
  flex-direction: column;
  gap: 24px;
  width: 100%;
}

form .alignItems {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 24px;
  width: 100%;
}

form a {
  color: rgba(255, 255, 255, 0.4);
  text-decoration: underline;
}

.column {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
}

.error {
  color: #d34848;
}

button {
  background-color: #403754;
  color: var(--white);
  border: none;
  border-radius: 4px;
  padding: 8px;
}

input {
  border: 1px solid rgba(255, 255, 255, 0.4);
  padding: 4px 8px;
  border-radius: 8px;
  color: var(--white);
}
</style>
