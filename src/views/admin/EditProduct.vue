<script setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import Navigation from "../../components/navComponent.vue";

// Bepaal de basis-URL op basis van de omgeving
const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

const router = useRouter();
const jwtToken = localStorage.getItem("jwtToken");

if (!jwtToken) {
  router.push("/login");
}

const route = useRoute();
const productId = ref(""); // Opslaan van de product ID
const productCode = ref("");
const productName = ref("");
const productPrice = ref(null);
const typeOfProduct = ref("sneaker");
const description = ref("");
const brand = ref("");
const colors = ref("");
const sizeOptions = ref([]);
const images = ref([]); // Voeg images toe
const lacesColor = ref([]);
const soleColor = ref([]);
const insideColor = ref([]);
const outsideColor = ref([]);

const productData = ref(null);

const fetchProductData = async () => {
  const id = route.params.id;
  productId.value = id;
  try {
    const response = await fetch(`${baseURL}/products/${id}`);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();
    productData.value = data.data.product;

    if (productData.value) {
      productCode.value = productData.value.productCode;
      productName.value = productData.value.productName;
      productPrice.value = productData.value.productPrice;
      typeOfProduct.value = productData.value.typeOfProduct;
      description.value = productData.value.description;
      brand.value = productData.value.brand;
      colors.value = productData.value.colors;
      sizeOptions.value = productData.value.sizeOptions;
      images.value = productData.value.images;
      lacesColor.value = productData.value.lacesColor;
      soleColor.value = productData.value.soleColor;
      insideColor.value = productData.value.insideColor;
      outsideColor.value = productData.value.outsideColor;
    }
  } catch (error) {
    console.error("Error fetching product data:", error);
    alert("Er is een fout opgetreden bij het ophalen van de productgegevens.");
  }
};

onMounted(() => {
  fetchProductData();
});

const handleFileChange = (event) => {
  // Verkrijg de geselecteerde bestanden
  images.value = Array.from(event.target.files);
};

const updateProduct = async () => {
  if (
    !productCode.value ||
    !productName.value ||
    !productPrice.value ||
    !description.value ||
    !brand.value ||
    !colors.value ||
    !sizeOptions.value ||
    !lacesColor.value ||
    !soleColor.value ||
    !insideColor.value ||
    !outsideColor.value
  ) {
    alert("Vul alstublieft alle vereiste velden in.");
    return;
  }

  const productDataToSend = {
    productCode: productCode.value,
    productName: productName.value,
    productPrice: productPrice.value,
    typeOfProduct: typeOfProduct.value,
    description: description.value,
    brand: brand.value,
    colors: colors.value,
    sizeOptions: sizeOptions.value,
    images: images.value, // Voeg images toe aan de request
    lacesColor: lacesColor.value,
    soleColor: soleColor.value,
    insideColor: insideColor.value,
    outsideColor: outsideColor.value,
  };

  try {
    const response = await fetch(`${baseURL}/products/${productId.value}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ product: productDataToSend }),
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const result = await response.json();
    router.push("/admin");
  } catch (error) {
    console.error("Error updating product:", error);
    alert("Er is een fout opgetreden bij het bijwerken van het product.");
  }
};
</script>

<template>
  <Navigation />
  <div class="content">
    <h1>Product Bewerken</h1>
    <form v-if="productData" @submit.prevent="updateProduct">
      <div class="row">
        <div class="column">
          <label for="productCode">Product Code:</label>
          <input v-model="productCode" id="productCode" type="text" required />
        </div>
        <div class="column">
          <label for="productName">Productnaam:</label>
          <input v-model="productName" id="productName" type="text" required />
        </div>
      </div>
      <div class="column">
        <label for="productPrice">Prijs:</label>
        <input
          v-model="productPrice"
          id="productPrice"
          type="number"
          required
        />
      </div>
      <div class="column">
        <label for="typeOfProduct">Type:</label>
        <input v-model="typeOfProduct" id="typeOfProduct" type="text" />
      </div>
      <div class="column">
        <label for="description">Beschrijving:</label>
        <input v-model="description" id="description" type="text" required />
      </div>
      <div class="column">
        <label for="brand">Merk:</label>
        <input v-model="brand" id="brand" type="text" required />
      </div>
      <div class="column">
        <label for="colors">Kleur:</label>
        <input v-model="colors" id="colors" type="text" />
      </div>
      <div class="column">
        <label for="sizeOptions">Maatopties:</label>
        <input v-model="sizeOptions" id="sizeOptions" type="text" />
      </div>
      <div class="column">
        <label for="lacesColor">Kleur Veters:</label>
        <input v-model="lacesColor" id="lacesColor" type="text" />
      </div>
      <div class="column">
        <label for="soleColor">Kleur Zool:</label>
        <input v-model="soleColor" id="soleColor" type="text" />
      </div>
      <div class="column">
        <label for="insideColor">Kleur Binnenkant:</label>
        <input v-model="insideColor" id="insideColor" type="text" />
      </div>
      <div class="column">
        <label for="outsideColor">Kleur Buitenkant:</label>
        <input v-model="outsideColor" id="outsideColor" type="text" />
      </div>
      <div class="column">
        <label for="images">Afbeeldingen:</label>
        <input @change="handleFileChange" id="images" type="file" multiple />
      </div>
      <button type="submit" class="btn active">Bewerk Product</button>
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
