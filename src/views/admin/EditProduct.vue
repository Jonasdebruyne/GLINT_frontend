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
const productId = ref < string > "";
const productCode = ref < string > "";
const typeOfProduct = ref < string > "optical";
const brand = ref < string > "";
const productName = ref < string > "";
const colors = ref < string > "";
const description = ref < string > "";
const glassColour = ref < string > "";
const activeUnactive = ref < string > "active";
const productPrice = (ref < number) | (null > null);
const sizeOptions = ref([]);
const material = ref < string > "";
const inStock = ref < boolean > true;
const lacesColor = ref([]);
const soleColor = ref([]);
const insideColor = ref([]);
const outsideColor = ref([]);

const productData = ref < any > null;

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
      typeOfProduct.value = productData.value.typeOfProduct;
      brand.value = productData.value.brand;
      productName.value = productData.value.productName;
      colors.value = productData.value.colors;
      description.value = productData.value.description;
      glassColour.value = productData.value.glassColour;
      activeUnactive.value = productData.value.activeUnactive;
      productPrice.value = productData.value.productPrice;
      sizeOptions.value = productData.value.sizeOptions;
      material.value = productData.value.material;
      inStock.value = productData.value.inStock;
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

const updateProduct = async () => {
  if (
    !productCode.value ||
    !brand.value ||
    !productName.value ||
    !activeUnactive.value ||
    !productPrice.value ||
    !sizeOptions.value ||
    !material.value ||
    !inStock.value ||
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
    typeOfProduct: typeOfProduct.value,
    brand: brand.value,
    productName: productName.value,
    colors: colors.value,
    description: description.value,
    glassColour: glassColour.value,
    activeUnactive: activeUnactive.value,
    productPrice: productPrice.value,
    sizeOptions: sizeOptions.value,
    material: material.value,
    inStock: inStock.value,
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
          <label for="brand">Merk:</label>
          <input v-model="brand" id="brand" type="text" required />
        </div>
      </div>
      <div class="column">
        <label for="productName">Productnaam:</label>
        <input v-model="productName" id="productName" type="text" required />
      </div>
      <div class="column">
        <label for="activeUnactive">Status:</label>
        <select v-model="activeUnactive" id="activeUnactive">
          <option value="active">Actief</option>
          <option value="inactive">Inactief</option>
        </select>
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
