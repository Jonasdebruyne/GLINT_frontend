<script lang="ts" setup>
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import Navigation from "../../components/navComponent.vue";

// Router en JWT token ophalen
const router = useRouter();
const jwtToken = localStorage.getItem("jwtToken");
const errorMessage = ref<string>("");

// Basis-URL bepalen afhankelijk van de omgeving
const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

// Functie om te controleren of de gebruiker is ingelogd
const checkToken = () => {
  if (!jwtToken) {
    router.push("/login");
  }
};

onMounted(() => {
  checkToken();
});

// Product-informatie refs
const productCode = ref<string>("");
const typeOfProduct = ref<string>("sneaker");
const productName = ref<string>("");
const brand = ref<string>("");
const productPrice = ref<number | null>(null);
const description = ref<string>("");
const colors = ref<string[]>([]); // Zorg ervoor dat kleuren altijd een array zijn
const sizeOptions = ref<string[]>([]); // Zorg ervoor dat maatopties altijd een array zijn
const lacesColor = ref<string[]>([]); // Zorg ervoor dat de schoenveterskleur altijd een array is
const soleColor = ref<string[]>([]); // Zorg ervoor dat de zoolkleur altijd een array is
const insideColor = ref<string[]>([]); // Zorg ervoor dat de binnenkleur altijd een array is
const outsideColor = ref<string[]>([]); // Zorg ervoor dat de buitenkleur altijd een array is

// Afbeeldingen initialiseren als een lege array
const images = ref<File[]>([]); // Dit zal de geüploade bestanden als File objecten opslaan

// Functie voor het verwerken van afbeelding upload naar Cloudinary
const uploadImageToCloudinary = async (file: File) => {
  const formData = new FormData();
  formData.append("file", file);
  formData.append("upload_preset", "ycy4zvmj"); // Vul hier je upload preset in
  formData.append("cloud_name", "dzempjvto"); // Vul hier je cloud name in

  try {
    console.log("Uploading file to Cloudinary:", file.name); // Debug: log de bestandsnaam
    const response = await fetch(
      `https://api.cloudinary.com/v1_1/dzempjvto/image/upload`,
      {
        method: "POST",
        body: formData,
      }
    );

    const data = await response.json();
    console.log("Cloudinary response data:", data); // Debug: log de response van Cloudinary

    if (response.ok && data.secure_url) {
      console.log("Image successfully uploaded:", data.secure_url); // Log de URL
      return data.secure_url; // Dit is de URL van de geüploade afbeelding
    } else {
      console.error("Cloudinary upload failed:", data);
      throw new Error(
        `Error uploading image to Cloudinary: ${data.error.message}`
      );
    }
  } catch (error) {
    console.error("Error uploading image:", error);
    throw error;
  }
};

// Functie voor het verwerken van bestandsevents
const handleImageUpload = (event: Event) => {
  const input = event.target as HTMLInputElement;
  if (input.files) {
    images.value = Array.from(input.files); // Voeg de bestanden toe aan de array
    console.log("Selected images:", images.value); // Log de geselecteerde bestanden
  }
};

// Functie om de invoer van kleurvelden om te zetten naar arrays
const parseColors = (input: string | string[]): string[] => {
  if (Array.isArray(input)) {
    return input; // Als het al een array is, doe verder niets
  }
  return input.split(",").map((color) => color.trim());
};

// Functie om het product toe te voegen
const addProduct = async () => {
  // Zorg ervoor dat de kleurvelden altijd arrays zijn
  const colorsArray = parseColors(colors.value);
  const sizeOptionsArray = parseColors(sizeOptions.value); // Maatopties ook als array
  const lacesColorArray = parseColors(lacesColor.value);
  const soleColorArray = parseColors(soleColor.value);
  const insideColorArray = parseColors(insideColor.value);
  const outsideColorArray = parseColors(outsideColor.value);

  // Controleer of de arrays gevuld zijn
  if (colorsArray.length === 0 || sizeOptionsArray.length === 0) {
    errorMessage.value =
      "Please provide at least one color and one size option.";
    return;
  }

  // Controleer of alle vereiste velden zijn ingevuld
  if (
    !productCode.value ||
    !productName.value ||
    !productPrice.value ||
    !description.value ||
    !brand.value ||
    colorsArray.length === 0 ||
    sizeOptionsArray.length === 0
  ) {
    errorMessage.value = "Please fill in all required fields.";
    return;
  }

  // Bereid het JSON-object voor in plaats van FormData
  const productData = {
    productCode: productCode.value,
    productName: productName.value,
    productPrice: productPrice.value,
    description: description.value,
    brand: brand.value,
    colors: colorsArray, // Colors as an array
    sizeOptions: sizeOptionsArray, // Size options as an array
    lacesColor: lacesColorArray, // Laces color as an array
    soleColor: soleColorArray, // Sole color as an array
    insideColor: insideColorArray, // Inside color as an array
    outsideColor: outsideColorArray, // Outside color as an array
  };

  // Voeg de afbeelding URL toe, door de afbeelding eerst naar Cloudinary te sturen
  if (images.value.length > 0) {
    try {
      console.log("Uploading images..."); // Debug: Begin met het uploaden van afbeeldingen
      const imageUrls = await Promise.all(
        images.value.map((file) => uploadImageToCloudinary(file)) // Upload elke afbeelding naar Cloudinary
      );
      console.log("Uploaded image URLs:", imageUrls); // Log de verkregen URLs van Cloudinary
      productData.images = imageUrls; // Voeg de Cloudinary-URL's toe aan de images array
    } catch (error) {
      errorMessage.value = "An error occurred while uploading images.";
      return;
    }
  } else {
    console.log("No images selected."); // Debug: Als geen afbeelding is geselecteerd
  }

  // Log de JSON voor debuggen
  console.log("Prepared Product Data:", productData);

  try {
    const response = await fetch(`${baseURL}/products`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json", // Verzend als JSON
        Authorization: `Bearer ${jwtToken}`,
      },
      body: JSON.stringify({ product: productData }), // Verzend het productobject als JSON
    });

    const data = await response.json(); // Parse de response body
    console.log("API Response:", data); // Log de volledige response body

    if (!response.ok) {
      // Als de response niet ok is, geef dan een gedetailleerde foutmelding
      errorMessage.value = `Error: ${data.message || "Unknown error"}`;
      return;
    }

    router.push("/admin"); // Redirect naar de admin pagina
  } catch (error) {
    console.error("Error adding product:", error);
    errorMessage.value = "An error occurred while adding the product.";
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
          <label for="productName">Product Name:</label>
          <input v-model="productName" id="productName" type="text" required />
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="typeOfProduct">Type Of Product:</label>
          <select v-model="typeOfProduct" id="typeOfProduct">
            <option value="sneaker">Sneaker</option>
            <option value="boot">Boot</option>
            <option value="sandals">Sandals</option>
            <option value="formal">Formal</option>
            <option value="slippers">Slippers</option>
          </select>
        </div>
        <div class="column">
          <label for="brand">Brand:</label>
          <input v-model="brand" id="brand" type="text" required />
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
            placeholder="Comma separated colors"
          />
        </div>
        <div class="column">
          <label for="sizeOptions">Size Options:</label>
          <input
            v-model="sizeOptions"
            id="sizeOptions"
            type="text"
            required
            placeholder="Comma separated sizes"
          />
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="productPrice">Price:</label>
          <input
            v-model.number="productPrice"
            id="price"
            type="number"
            required
            min="0"
            step="0.01"
          />
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="lacesColor">Laces Color:</label>
          <input type="color" v-model="lacesColor" class="colorpicker" />
        </div>
        <div class="column">
          <label for="soleColor">Sole Color:</label>
          <input type="color" v-model="soleColor" class="colorpicker" />
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="insideColor">Inside Color:</label>
          <input type="color" v-model="insideColor" class="colorpicker" />
        </div>
        <div class="column">
          <label for="outsideColor">Outside Color:</label>
          <input type="color" v-model="outsideColor" class="colorpicker" />
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="description">Description:</label>
          <textarea
            v-model="description"
            id="description"
            required
            placeholder="Enter product description"
          ></textarea>
        </div>
      </div>

      <div>
        <label for="images">Product Images:</label>
        <input type="file" id="images" multiple @change="handleImageUpload" />
      </div>

      <button type="submit">Add Product</button>
      <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
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

.error {
  color: #d34848;
}
</style>
