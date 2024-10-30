<script lang="ts" setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import Navigation from "../../components/navComponent.vue";

const router = useRouter();
const jwtToken = localStorage.getItem("jwtToken");
const errorMessage = ref<string>("");

// Basis-URL bepalen afhankelijk van de omgeving
const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

const checkToken = () => {
  if (!jwtToken) {
    router.push("/login");
  }
};

onMounted(() => {
  checkToken();
});

const productCode = ref<string>("");
const typeOfProduct = ref<string>("optical");
const brand = ref<string>("");
const productName = ref<string>("");
const colors = ref<string>("");
const description = ref<string>("");
const glassColor = ref<string>("");
const status = ref<string>("active");

const setglassColor = (color: string) => {
  glassColor.value = color;
};

const setColor = (color: string) => {
  colors.value = color;
};

const addProduct = async () => {
  if (
    !productCode.value ||
    !typeOfProduct.value ||
    !brand.value ||
    !productName.value ||
    !colors.value ||
    !description.value ||
    !glassColor.value ||
    !status.value
  ) {
    errorMessage.value = "Vul alle velden in.";
    return;
  }

  try {
    const response = await fetch(`${baseURL}/products`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        product: {
          productCode: productCode.value,
          typeOfProduct: typeOfProduct.value,
          brand: brand.value,
          productName: productName.value,
          colors: colors.value,
          description: description.value,
          glassColor: glassColor.value,
          activeUnactive: status.value,
        },
      }),
    });

    if (!response.ok) {
      const errorData = await response.json();
      console.error("Error response:", errorData);
      errorMessage.value = "Er is een fout opgetreden: " + errorData.message;
      return;
    }

    await response.json();
    router.push("/admin");
  } catch (error) {
    console.error("Error adding product:", error);
    errorMessage.value =
      "Er is een fout opgetreden bij het toevoegen van het product.";
  }
};
</script>

<template>
  <Navigation />
  <div class="content">
    <h1>Add New Product</h1>
    <form @submit.prevent="addProduct">
      <div class="row">
        <div class="column">
          <label for="productCode">Product Code:</label>
          <input v-model="productCode" id="productCode" type="text" required />
        </div>
        <div class="column">
          <label for="typeOfProduct">Type Of Product:</label>
          <select v-model="typeOfProduct" id="typeOfProduct">
            <option value="optical">Optical</option>
            <option value="sun">Sun</option>
          </select>
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="brand">Brand:</label>
          <input v-model="brand" id="brand" type="text" required />
        </div>
        <div class="column">
          <label for="productName">Product Name:</label>
          <input v-model="productName" id="productName" type="text" required />
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="colors">Colors:</label>
          <input
            v-model="colors"
            id="colors"
            type="text"
            required
            placeholder="#000000 or rgb(0, 0, 0)"
          />
          <input
            type="color"
            v-model="colors"
            class="colorpicker"
            @input="setColor($event.target.value)"
          />
        </div>
        <div class="column">
          <label for="description">Description:</label>
          <textarea v-model="description" id="description" required></textarea>
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="glassColor">Glass Colour:</label>
          <input
            v-model="glassColor"
            id="glassColor"
            type="text"
            required
            placeholder="#000000 or rgb(0, 0, 0)"
          />
          <input
            class="colorpicker"
            type="color"
            v-model="glassColor"
            @input="setglassColor($event.target.value)"
          />
        </div>
        <div class="column">
          <label for="status">Status:</label>
          <select v-model="status" id="status">
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
          </select>
        </div>
      </div>

      <div v-if="errorMessage" class="error">{{ errorMessage }}</div>

      <button type="submit" class="btn active">Add Product</button>
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
  gap: 20px;
  width: 100%;
}

form .column {
  display: flex;
  flex-direction: column;
  gap: 8px;
  width: 100%;
}

input,
select,
textarea {
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #333;
  background-color: #333;
  color: white;
  width: 100%;
}

button {
  color: white;
  cursor: pointer;
}

.colorpicker {
  padding: 0;
  width: 32px;
  height: 32px;
  background-color: transparent;
  border: none;
}
</style>
