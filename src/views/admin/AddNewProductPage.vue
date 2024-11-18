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
const status = ref<string>("active");
const description = ref<string>("");

// Shoe-specific fields
const sizeOptions = ref<string>("");
const colors = ref<string>("");
const lacesColor = ref<string>("");
const soleColor = ref<string>("");
const insideColor = ref<string>("");
const outsideColor = ref<string>("");

// Afbeeldingen initialiseren als een lege array
const images = ref<string[]>([]); // Zorg ervoor dat dit een lege array is

// Kleurinstellingsfuncties
const setColor = (color: string) => {
  colors.value = color;
};
const setLacesColor = (color: string) => {
  lacesColor.value = color;
};
const setSoleColor = (color: string) => {
  soleColor.value = color;
};
const setInsideColor = (color: string) => {
  insideColor.value = color;
};
const setOutsideColor = (color: string) => {
  outsideColor.value = color;
};

// Functie voor het toevoegen van afbeeldingen (uploaden van bestanden)
const handleImageUpload = (event: Event) => {
  const files = (event.target as HTMLInputElement).files;
  if (files && files.length > 0) {
    // Converteer bestanden naar een array van URL's (of upload naar een server en haal URL's terug)
    images.value = Array.from(files).map((file) => URL.createObjectURL(file));
  }
};

// Functie om het product toe te voegen
const addProduct = async () => {
  if (
    !productCode.value ||
    !typeOfProduct.value ||
    !productName.value ||
    !brand.value ||
    !productPrice.value ||
    !status.value ||
    !description.value ||
    !sizeOptions.value ||
    !colors.value ||
    !lacesColor.value ||
    !soleColor.value ||
    !insideColor.value ||
    !outsideColor.value ||
    images.value.length === 0 // Controleer of er afbeeldingen zijn toegevoegd
  ) {
    errorMessage.value = "Vul alle velden in.";
    return;
  }

  try {
    // Verstuur het product naar de backend via een POST request
    const response = await fetch(`${baseURL}/products`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        product: {
          productCode: productCode.value,
          typeOfProduct: typeOfProduct.value,
          productName: productName.value,
          brand: brand.value,
          productPrice: productPrice.value,
          activeUnactive: status.value,
          description: description.value,
          sizeOptions: sizeOptions.value,
          colors: colors.value,
          lacesColor: lacesColor.value,
          soleColor: soleColor.value,
          insideColor: insideColor.value,
          outsideColor: outsideColor.value,
          images: images.value, // Stuur de lijst met afbeeldingen
        },
      }),
    });

    if (!response.ok) {
      const errorData = await response.json();
      console.error("Error response:", errorData);
      errorMessage.value = "Er is een fout opgetreden: " + errorData.message;
      return;
    }

    // Redirect naar de admin-pagina na succesvolle toevoeging
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
    <h1>Add New Shoe</h1>
    <form @submit.prevent="addProduct">
      <div class="row">
        <div class="column">
          <label for="productCode">Product Code:</label>
          <input v-model="productCode" id="productCode" type="text" required />
        </div>
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
      </div>

      <div class="row">
        <div class="column">
          <label for="productName">Product Name:</label>
          <input v-model="productName" id="productName" type="text" required />
        </div>
        <div class="column">
          <label for="brand">Brand:</label>
          <input v-model="brand" id="brand" type="text" required />
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="sizeOptions">Size Options:</label>
          <input
            v-model="sizeOptions"
            id="sizeOptions"
            type="text"
            required
            placeholder="e.g., 39, 40, 41, 42"
          />
        </div>
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
          <label for="status">Status:</label>
          <select v-model="status" id="status">
            <option value="active">Active</option>
            <option value="inactive">Inactive</option>
          </select>
        </div>
        <div class="column">
          <label for="description">Description:</label>
          <textarea v-model="description" id="description" required></textarea>
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="colors">Primary Color:</label>
          <input
            type="color"
            v-model="colors"
            class="colorpicker"
            @input="setColor($event.target.value)"
          />
        </div>
        <div class="column">
          <label for="lacesColor">Laces Color:</label>
          <input
            type="color"
            v-model="lacesColor"
            class="colorpicker"
            @input="setLacesColor($event.target.value)"
          />
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="soleColor">Sole Color:</label>
          <input
            type="color"
            v-model="soleColor"
            class="colorpicker"
            @input="setSoleColor($event.target.value)"
          />
        </div>
        <div class="column">
          <label for="insideColor">Inside Color:</label>
          <input
            type="color"
            v-model="insideColor"
            class="colorpicker"
            @input="setInsideColor($event.target.value)"
          />
        </div>
      </div>

      <div class="row">
        <div class="column">
          <label for="outsideColor">Outside Color:</label>
          <input
            type="color"
            v-model="outsideColor"
            class="colorpicker"
            @input="setOutsideColor($event.target.value)"
          />
        </div>
        <div class="column">
          <label for="images">Images:</label>
          <input
            id="images"
            type="file"
            accept="image/*"
            multiple
            @change="handleImageUpload"
          />
          <div v-if="images.value && images.value.length > 0">
            <p>Images Preview:</p>
            <div v-for="(image, index) in images.value" :key="index">
              <img :src="image" alt="Image preview" width="100" />
            </div>
          </div>
        </div>
      </div>

      <p v-if="errorMessage" class="error">{{ errorMessage }}</p>

      <button type="submit" class="btn active">Add Shoe</button>
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





<!-- <script lang="ts" setup>
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
const typeOfProduct = ref<string>("optical");
const productName = ref<string>("");
const brand = ref<string>("");
const colors = ref<string>("");
const glassColor = ref<string>("");
const productPrice = ref<number | null>(null);
const status = ref<string>("active");
const description = ref<string>("");

// Afbeeldingen initialiseren als een lege array
const images = ref<string[]>([]); // Zorg ervoor dat dit een lege array is

// Functie voor het instellen van de glazenkleur
const setglassColor = (color: string) => {
  glassColor.value = color;
};

// Functie voor het instellen van de kleur
const setColor = (color: string) => {
  colors.value = color;
};

// Functie voor het toevoegen van afbeeldingen (uploaden van bestanden)
const handleImageUpload = (event: Event) => {
  const files = (event.target as HTMLInputElement).files;
  if (files && files.length > 0) {
    // Converteer bestanden naar een array van URL's (of upload naar een server en haal URL's terug)
    images.value = Array.from(files).map((file) => URL.createObjectURL(file));
  }
};

// Functie om het product toe te voegen
const addProduct = async () => {
  if (
    !productCode.value ||
    !typeOfProduct.value ||
    !productName.value ||
    !brand.value ||
    !colors.value ||
    !glassColor.value ||
    !productPrice.value ||
    !status.value ||
    !description.value ||
    images.value.length === 0 // Controleer of er afbeeldingen zijn toegevoegd
  ) {
    errorMessage.value = "Vul alle velden in.";
    return;
  }

  try {
    // Verstuur het product naar de backend via een POST request
    const response = await fetch(`${baseURL}/products`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        product: {
          productCode: productCode.value,
          typeOfProduct: typeOfProduct.value,
          productName: productName.value,
          brand: brand.value,
          colors: colors.value,
          glassColor: glassColor.value,
          productPrice: productPrice.value,
          activeUnactive: status.value,
          description: description.value,
          images: images.value, // Stuur de lijst met afbeeldingen
        },
      }),
    });

    if (!response.ok) {
      const errorData = await response.json();
      console.error("Error response:", errorData);
      errorMessage.value = "Er is een fout opgetreden: " + errorData.message;
      return;
    }

    // Redirect naar de admin-pagina na succesvolle toevoeging
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
          <label for="productName">Product Name:</label>
          <input v-model="productName" id="productName" type="text" required />
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
      </div>

      <div class="row">
        <div class="column">
          <label for="productPrice">Prijs:</label>
          <input
            v-model.number="productPrice"
            id="price"
            type="number"
            required
            min="0"
            step="0.01"
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

      <div class="row">
        <div class="column">
          <label for="description">Description:</label>
          <textarea v-model="description" id="description" required></textarea>
        </div>
        <div class="column">
          <label for="images">Images:</label>
          <input
            id="images"
            type="file"
            accept="image/*"
            multiple
            @change="handleImageUpload"
          />
          <div v-if="images.value && images.value.length > 0">
            <p>Images Preview:</p>
            <div v-for="(image, index) in images.value" :key="index">
              <img :src="image" alt="Image preview" width="100" />
            </div>
          </div>
        </div>
      </div>

      <p v-if="errorMessage" class="error">{{ errorMessage }}</p>

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

.error {
  color: #d34848;
}
</style> -->
