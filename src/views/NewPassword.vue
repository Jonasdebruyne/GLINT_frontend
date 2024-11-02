<script lang="ts" setup>
import { ref, onMounted } from "vue";
import { useRouter, useRoute } from "vue-router";
import axios from "axios";

const verificationCode = ref("");
const errorMessage = ref("");
const router = useRouter();
const route = useRoute(); // Voor toegang tot routeparameters

const email = ref(""); // Hier slaan we het emailadres op

// Functie om te controleren of de code geldig is
const isValidCode = (code: string) => {
  return code.trim() !== ""; // Zorg ervoor dat de code niet leeg is
};

// Haal het emailadres op bij het laden van het component
onMounted(() => {
  if (route.query.email) {
    email.value = route.query.email as string;
    console.log("Email opgehaald van URL:", email.value);
  } else {
    console.log("Geen email gevonden in de URL.");
  }
});

// Functie om de verificatiecode te versturen
const verifyCode = async () => {
  console.log("Verificatiecode ingevoerd:", verificationCode.value);

  if (!isValidCode(verificationCode.value)) {
    errorMessage.value = "Voer een geldige verificatiecode in.";
    console.log("Ongeldige code ingevoerd.");
    return;
  }

  try {
    console.log("Verificatiecode en email worden verzonden naar de server...");
    const response = await axios.post(
      "http://localhost:3000/api/v1/users/verify-code",
      {
        code: verificationCode.value,
        email: email.value,
      }
    );

    console.log("Serverrespons ontvangen:", response.status);
    if (response.status === 200) {
      console.log(
        "Verificatiecode correct, navigeren naar nieuwe wachtwoordpagina."
      );
      router.push("/newPassword"); // Navigatie naar de nieuwe wachtwoordpagina
    } else {
      errorMessage.value = "De verificatiecode is ongeldig.";
      console.log("Ongeldige verificatiecode van de server ontvangen.");
    }
  } catch (error) {
    console.log("Fout bij het verifiëren van de code:", error);
    if (axios.isAxiosError(error) && error.response) {
      // Specifieke foutafhandeling op basis van de statuscode
      switch (error.response.status) {
        case 400:
          errorMessage.value = "Ongeldige verificatiecode.";
          break;
        case 404:
          errorMessage.value = "Verificatiecode niet gevonden.";
          break;
        default:
          errorMessage.value =
            "Er is een fout opgetreden bij het verifiëren van de code.";
          break;
      }
      console.log("Serverfout:", error.response.status, error.response.data);
    } else {
      errorMessage.value = "Er is een netwerkfout opgetreden.";
      console.log("Netwerkfout of andere fout:", error);
    }
  }
};
</script>

<template>
  <div class="container">
    <div class="overlay">
      <div class="elements">
        <h1>Forgot password</h1>
        <form @submit.prevent="verifyCode">
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
/* Jouw bestaande CSS-stijlen */
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
  color: var(--white);
}
</style>
