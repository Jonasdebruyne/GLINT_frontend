<script lang="ts" setup>
import { ref, watch, onMounted } from "vue";
import { useRoute } from "vue-router";
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { GUI } from "dat.gui";

const lacesColors = ref<string[]>([]);
const solesColors = ref<string[]>([]);
const insideColors = ref<string[]>([]);
const outsideColors = ref<string[]>([]);

const selectedColor = ref<string | null>(null);
const selectedLacesColor = ref<string | null>(null);
const selectedSoleColor = ref<string | null>(null);
const selectedInsideColor = ref<string | null>(null);
const selectedOutsideColor = ref<string | null>(null);

function selectColorForLaces(color: string) {
  selectedColor.value = color;
  selectedLacesColor.value = color; // Bewaar de geselecteerde kleur
  if (window.laces) {
    window.laces.material.color.set(color);
  }
}

function selectColorForSole(color: string) {
  selectedColor.value = color;
  selectedSoleColor.value = color; // Bewaar de geselecteerde kleur
  if (window.sole && window.sole.material) {
    window.sole.material.color.set(color);
  } else {
    console.error("Sole object or its material not found");
  }
}

function selectColorForInside(color: string) {
  selectedColor.value = color;
  selectedInsideColor.value = color; // Bewaar de geselecteerde kleur
  if (window.inside && window.inside.material) {
    window.inside.material.color.set(color);
  } else {
    console.error("Inside object or its material not found");
  }
}

function selectColorForOutside(color: string) {
  selectedColor.value = color;
  selectedOutsideColor.value = color; // Bewaar de geselecteerde kleur
  if (window.outside && window.outside.material) {
    window.outside.material.color.set(color);
  } else {
    console.error("Outside object or its material not found");
  }
}

const route = useRoute();
const productCode = ref<string | undefined>(undefined);
const productData = ref<{ productName: string; productPrice: number } | null>(
  null
);
const isLoading = ref<boolean>(true);
const error = ref<string | null>(null);

watch(
  () => route.params.productCode,
  (newCode) => {
    if (newCode) {
      productCode.value = newCode as string;
      fetchProductData(newCode);
    } else {
      console.error("Product code is undefined");
    }
  },
  { immediate: true }
);

onMounted(() => {
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  camera.position.set(0, 5, 15);
  camera.lookAt(0, 0, 0);

  const renderer = new THREE.WebGLRenderer();
  const container = document.querySelector(".model") as HTMLElement;
  renderer.setSize(container.offsetWidth, container.offsetHeight);
  container.appendChild(renderer.domElement);

  // Add an environment map
  const envTextureLoader = new THREE.CubeTextureLoader();
  const environmentMap = envTextureLoader.load([
    "/textures/px.png",
    "/textures/nx.png",
    "/textures/py.png",
    "/textures/ny.png",
    "/textures/pz.png",
    "/textures/nz.png",
  ]);
  scene.background = environmentMap;
  scene.environment = environmentMap;

  const light = new THREE.PointLight(0xffffff, 1, 100);
  light.position.set(10, 10, 10);
  scene.add(light);

  const directionalLight = new THREE.DirectionalLight(0xffffff, 0.4);
  directionalLight.position.set(10, 20, 10);
  scene.add(directionalLight);

  const ambientLight = new THREE.AmbientLight(0x404040);
  scene.add(ambientLight);

  const gltfLoader = new GLTFLoader();
  const dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath("/node_modules/three/examples/jsm/libs/draco/");
  gltfLoader.setDRACOLoader(dracoLoader);

  gltfLoader.load(
    "/models/Shoe_compressed.glb",
    (gltf) => {
      console.log("Loaded GLB model scene:", gltf.scene);

      // Het model laden zoals je al deed
      gltf.scene.scale.set(50, 50, 50);
      gltf.scene.position.set(0, 0, 0);
      scene.add(gltf.scene);

      // Traverse door het model en zoek specifieke onderdelen
      gltf.scene.traverse((child) => {
        switch (child.name) {
          case "laces":
            window.laces = child;
            break;
          case "sole_bottom":
            window.sole = child;
            break;
          case "inside":
            window.inside = child;
            break;
          case "outside_1":
            window.outside = child;
            break;
        }
      });

      console.log("Children in GLB scene:", gltf.scene.children);

      // Setup OrbitControls en GUI zoals eerder

      animate();
    },
    // Voortgangsfunctie
    (xhr) => {
      const progress = (xhr.loaded / xhr.total) * 100;
      if (xhr.total !== 0) {
        console.log(`Model geladen: ${progress.toFixed(2)}%`);
      } else {
        console.log("Model geladen (onbekende voortgang)");
      }
    },
    // Foutmelding bij het laden
    (error) => {
      console.error("Error loading GLB file:", error);
    }
  );

  function animate() {
    requestAnimationFrame(animate);

    if (scene.children.length > 0) {
      const model = scene.children[0];
      model.rotation.y += 0.01;
    }

    renderer.render(scene, camera);
  }

  window.addEventListener("resize", () => {
    renderer.setSize(container.offsetWidth, container.offsetHeight);
    camera.aspect = container.offsetWidth / container.offsetHeight;
    camera.updateProjectionMatrix();
  });

  animate();
});

async function fetchProductData(code: string) {
  isLoading.value = true;
  error.value = null;

  try {
    const response = await fetch(
      `http://localhost:3000/api/v1/products/${code}`
    );
    if (!response.ok) throw new Error("Network response was not ok");

    const data = await response.json();
    productData.value = {
      productName: data.data.product.productName,
      productPrice: data.data.product.productPrice,
    };

    lacesColors.value = data.data.product.lacesColor || [];
    solesColors.value = data.data.product.soleColor || [];
    insideColors.value = data.data.product.insideColor || [];
    outsideColors.value = data.data.product.outsideColor || [];
  } catch (err) {
    console.error("Error occurred:", err);
    error.value = "Unable to fetch product information.";
  } finally {
    isLoading.value = false;
  }
}

onMounted(() => {
  const listItems = document.querySelectorAll(".overview ul li");
  const pages = document.querySelectorAll(".config-ui__page");
  const overview = document.querySelector(".overview");
  const summary = document.querySelector(".summary");
  const overviewButton = document.querySelector(
    ".config-wrapper .overviewButton"
  );
  const summaryButton = document.querySelector(
    ".config-wrapper .summaryButton"
  );
  const backButton = document.querySelector(".config-wrapper .backButton");
  const nextButton = document.querySelector(".config-wrapper .nextButton");

  let currentPageIndex = -1;

  function updateButtonVisibility() {
    backButton?.style.setProperty(
      "visibility",
      currentPageIndex === -1 ? "hidden" : "visible"
    );

    nextButton?.style.setProperty(
      "visibility",
      currentPageIndex >= pages.length - 1 ? "hidden" : "visible"
    );

    overviewButton?.style.setProperty(
      "visibility",
      currentPageIndex === -1 ? "hidden" : "visible"
    );

    summaryButton?.style.setProperty(
      "visibility",
      currentPageIndex === pages.length ? "hidden" : "visible"
    );
  }

  listItems.forEach((li, index) => {
    li.addEventListener("click", () => {
      if (overview) {
        overview.style.display = "none";
      }

      pages.forEach((page) => (page.style.display = "none"));
      if (pages[index]) {
        pages[index].style.display = "flex";
      }

      currentPageIndex = index;
      updateButtonVisibility();
    });
  });

  nextButton?.addEventListener("click", () => {
    if (currentPageIndex === -1) {
      currentPageIndex = 0;
      overview.style.display = "none";
      pages[currentPageIndex]?.style.setProperty("display", "flex");
    } else if (currentPageIndex < pages.length - 1) {
      pages[currentPageIndex]?.style.setProperty("display", "none");
      currentPageIndex++;
      pages[currentPageIndex]?.style.setProperty("display", "flex");
    }
    updateButtonVisibility();
  });

  backButton?.addEventListener("click", () => {
    if (currentPageIndex === pages.length) {
      summary.style.display = "none";
      currentPageIndex = pages.length - 1;
      pages[currentPageIndex]?.style.setProperty("display", "flex");
    } else if (currentPageIndex > 0) {
      pages[currentPageIndex]?.style.setProperty("display", "none");
      currentPageIndex--;
      pages[currentPageIndex]?.style.setProperty("display", "flex");
    } else if (currentPageIndex === 0) {
      currentPageIndex = -1;
      overview.style.display = "flex";
      pages.forEach((page) => (page.style.display = "none"));
    }
    updateButtonVisibility();
  });

  overviewButton?.addEventListener("click", () => {
    currentPageIndex = -1;
    overview.style.display = "flex";
    summary.style.display = "none";
    pages.forEach((page) => (page.style.display = "none"));
    updateButtonVisibility();
  });

  summaryButton?.addEventListener("click", () => {
    currentPageIndex = pages.length;
    overview.style.display = "none";
    summary.style.display = "flex";
    pages.forEach((page) => (page.style.display = "none"));
    updateButtonVisibility();
  });

  if (currentPageIndex === -1) {
    overview.style.display = "flex";
    pages.forEach((page) => (page.style.display = "none"));
    backButton?.style.setProperty("visibility", "hidden");
    nextButton?.style.setProperty("visibility", "visible");
  }

  updateButtonVisibility();
});

function validateColors(): boolean {
  return (
    selectedLacesColor.value &&
    selectedSoleColor.value &&
    selectedInsideColor.value &&
    selectedOutsideColor.value
  );
}

const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

async function submitOrder() {
  if (!validateColors()) {
    const errorMessageElement = document.querySelector(".errorMessage");
    if (errorMessageElement) {
      errorMessageElement.innerHTML = "Please select a color for every part!";
    }
    return;
  }

  const orderData = {
    productCode: productCode.value,
    lacesColor: selectedLacesColor.value,
    soleColor: selectedSoleColor.value,
    insideColor: selectedInsideColor.value,
    outsideColor: selectedOutsideColor.value,
  };

  console.log("Submitting order data:", orderData);

  try {
    const response = await fetch(`${baseURL}/products`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(orderData),
    });

    if (!response.ok) {
      const errorResponse = await response.json();
      console.error("Server error response:", errorResponse);
      throw new Error(errorResponse.message || "Failed to submit the order");
    }

    const result = await response.json();
    console.log("Order submitted successfully:", result);

    document.querySelector(".errorMessage").innerHTML = "";

    document.querySelector(".successMessage").innerHTML =
      "Order submitted successfully!";
  } catch (error) {
    console.error("Error submitting order:", error);

    const errorMessageElement = document.querySelector(".errorMessage");
    if (errorMessageElement) {
      errorMessageElement.innerHTML =
        "There was an error submitting your order. Please try again.";
    }
  }
}
</script>

<template>
  <div class="container">
    <h3 class="logo">REBILT</h3>
    <div class="model"></div>
    <div class="icons">
      <router-link :to="`/`">
        <div class="icon">
          <div>
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="26"
              height="26"
              viewBox="0 0 256 256"
            >
              <path
                d="M104,216V152h48v64h64V120a8,8,0,0,0-2.34-5.66l-80-80a8,8,0,0,0-11.32,0l-80,80A8,8,0,0,0,40,120v96Z"
                fill="none"
                stroke="white"
                stroke-width="8"
              ></path>
            </svg>
          </div>
          <p>Home</p>
        </div>
      </router-link>
      <div class="icon">
        <div>
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
            <path
              d="M11.98 2.36a.75.75 0 00-.33.1L8.3 4.11H8.3a.08.08 0 00-.01.01.76.76 0 00-.31 1c.09.18.24.31.43.38.19.06.4.05.57-.04l2.26-1.13v2.7c0 .2.08.39.22.53a.76.76 0 001.08 0 .76.76 0 00.22-.54V4.34L15 5.47h.01a.76.76 0 001.03-.91.75.75 0 00-.34-.42l-.01-.01a.08.08 0 00-.01 0l-3.34-1.68a.75.75 0 00-.37-.09zm-5.57 2.8a.76.76 0 00-.34.08L2.72 6.9a.76.76 0 00-.42.7v3.9c0 .2.08.39.22.53a.76.76 0 001.07 0 .76.76 0 00.22-.54V8.81l2.26 1.13a.76.76 0 001.05-.91.76.76 0 00-.35-.42.08.08 0 00-.02-.02l-2-1 2-1c.3-.16.48-.52.4-.85a.77.77 0 00-.74-.58zm11.18 0a.77.77 0 00-.73.58c-.08.32.08.67.37.83l.02.02 2 1-2 1a.08.08 0 00-.02.02c-.17.1-.3.24-.35.42a.76.76 0 00.47.95c.19.06.4.05.58-.04l2.26-1.13v2.7c0 .2.08.39.22.53a.76.76 0 001.07 0 .76.76 0 00.22-.54V7.6a.76.76 0 00-.42-.69l-3.35-1.67a.76.76 0 00-.34-.08zm-2.24 4.47a.75.75 0 00-.33.08L12 11.2l-3.02-1.5a.76.76 0 00-1.05.91c.06.18.2.33.36.42a.08.08 0 000 .01h.01a.08.08 0 000 .01l2.94 1.47v2.89c0 .2.08.39.22.53a.76.76 0 001.08 0 .76.76 0 00.22-.53v-2.9l2.93-1.46.02-.02c.3-.16.45-.5.37-.83a.77.77 0 00-.73-.58zM3.01 14.11a.75.75 0 00-.7.75v3.9c-.01.29.15.56.41.69l3.35 1.67a.76.76 0 001.05-.91.76.76 0 00-.34-.42.08.08 0 00-.03-.02l-2-1 2-1 .02-.02c.17-.1.3-.24.35-.42a.76.76 0 00-.47-.95.77.77 0 00-.58.04l-2.26 1.13v-2.7c0-.2-.08-.4-.24-.55a.76.76 0 00-.56-.2zm17.88 0a.76.76 0 00-.7.75v2.69l-2.26-1.13a.76.76 0 00-1.05.91c.06.18.18.33.35.42a.08.08 0 00.01 0v.02a.08.08 0 00.01 0l2 1-2 1h-.01a.08.08 0 000 .01h-.01a.76.76 0 00.13 1.37c.18.07.39.06.57-.03l3.35-1.67c.26-.13.42-.4.42-.69v-3.9a.76.76 0 00-.8-.75zm-8.94 4.47a.08.08 0 00-.03 0 .75.75 0 00-.47.23.76.76 0 00-.2.52v2.69l-2.26-1.13a.08.08 0 00-.01 0 .76.76 0 00-1.03.91c.05.18.18.33.34.42l.01.01a.08.08 0 00.01.01l3.34 1.67c.22.12.49.11.7 0l3.34-1.67.02-.01a.76.76 0 00.31-1 .76.76 0 00-1-.34l-2.26 1.13v-2.7a.76.76 0 00-.8-.74z"
            ></path>
          </svg>
        </div>
        <p>AR</p>
      </div>
    </div>
    <div class="rotate-informer">
      <svg
        xmlns="http://www.w3.org/2000/svg"
        width="24"
        height="24"
        fill="#ffffff"
        viewBox="0 0 256 256"
      >
        <path
          d="M232,48V88a8,8,0,0,1-16,0V56H184a8,8,0,0,1,0-16h40A8,8,0,0,1,232,48ZM72,200H40V168a8,8,0,0,0-16,0v40a8,8,0,0,0,8,8H72a8,8,0,0,0,0-16Zm152-40a8,8,0,0,0-8,8v32H184a8,8,0,0,0,0,16h40a8,8,0,0,0,8-8V168A8,8,0,0,0,224,160ZM32,96a8,8,0,0,0,8-8V56H72a8,8,0,0,0,0-16H32a8,8,0,0,0-8,8V88A8,8,0,0,0,32,96ZM188,167l-56,32a8,8,0,0,1-7.94,0L68,167A8,8,0,0,1,64,160V96a8,8,0,0,1,4-7l56-32a8,8,0,0,1,7.94,0l56,32a8,8,0,0,1,4,7v64A8,8,0,0,1,188,167ZM88.12,96,128,118.79,167.88,96,128,73.21ZM80,155.36l40,22.85V132.64L80,109.79Zm96,0V109.79l-40,22.85v45.57Z"
        ></path>
      </svg>
      <p>Drag to rotate</p>
    </div>
    <div class="config-wrapper">
      <div class="links">
        <a href="#" class="overviewButton visibility">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            viewBox="0 0 320 512"
            style="transform: rotate(180deg)"
          >
            <path
              d="M310.6 233.4c12.5 12.5 12.5 32.8 0 45.3l-192 192c-12.5 12.5-32.8 12.5-45.3 0s-12.5-32.8 0-45.3L242.7 256 73.4 86.6c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0l192 192z"
            ></path>
          </svg>
          <p>Overview</p>
        </a>
        <a href="#" class="summaryButton">
          <p>Summary</p>
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512">
            <path
              d="M310.6 233.4c12.5 12.5 12.5 32.8 0 45.3l-192 192c-12.5 12.5-32.8 12.5-45.3 0s-12.5-32.8 0-45.3L242.7 256 73.4 86.6c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0l192 192z"
            ></path>
          </svg>
        </a>
      </div>
      <div class="elements">
        <div class="overview">
          <h2>Overview</h2>
          <ul>
            <li>
              <p>1. Choose the color of the laces</p>
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512">
                <path
                  d="M310.6 233.4c12.5 12.5 12.5 32.8 0 45.3l-192 192c-12.5 12.5-32.8 12.5-45.3 0s-12.5-32.8 0-45.3L242.7 256 73.4 86.6c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0l192 192z"
                ></path>
              </svg>
            </li>
            <li>
              <p>2. Choose the color of the sole</p>
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512">
                <path
                  d="M310.6 233.4c12.5 12.5 12.5 32.8 0 45.3l-192 192c-12.5 12.5-32.8 12.5-45.3 0s-12.5-32.8 0-45.3L242.7 256 73.4 86.6c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0l192 192z"
                ></path>
              </svg>
            </li>
            <li>
              <p>3. Choose the color of the inside of your shoe</p>
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512">
                <path
                  d="M310.6 233.4c12.5 12.5 12.5 32.8 0 45.3l-192 192c-12.5 12.5-32.8 12.5-45.3 0s-12.5-32.8 0-45.3L242.7 256 73.4 86.6c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0l192 192z"
                ></path>
              </svg>
            </li>
            <li>
              <p>4. Choose the color of the outside of your shoe</p>
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512">
                <path
                  d="M310.6 233.4c12.5 12.5 12.5 32.8 0 45.3l-192 192c-12.5 12.5-32.8 12.5-45.3 0s-12.5-32.8 0-45.3L242.7 256 73.4 86.6c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0l192 192z"
                ></path>
              </svg>
            </li>
          </ul>
        </div>
        <div class="config-ui__page page1 colorsItem display">
          <h2>Choose the color of the laces</h2>
          <div class="row">
            <div
              v-for="color in lacesColors"
              :key="color"
              :class="{ active: selectedColor === color }"
              @click="selectColorForLaces(color)"
              :style="{ backgroundColor: color }"
            ></div>
          </div>
        </div>
        <div class="config-ui__page page2 colorsItem">
          <h2>Choose the color of the sole</h2>
          <div class="row">
            <div
              v-for="color in solesColors"
              :key="color"
              :class="{ active: selectedColor === color }"
              @click="selectColorForSole(color)"
              :style="{ backgroundColor: color }"
            ></div>
          </div>
        </div>
        <div class="config-ui__page page3 colorsItem">
          <h2>Choose the color of the inside of your shoe</h2>
          <div class="row">
            <div
              v-for="color in insideColors"
              :key="color"
              :class="{ active: selectedColor === color }"
              @click="selectColorForInside(color)"
              :style="{ backgroundColor: color }"
            ></div>
          </div>
        </div>
        <div class="config-ui__page page4 colorsItem">
          <h2>Choose the color of the outside of your shoe</h2>
          <div class="row">
            <div
              v-for="color in outsideColors"
              :key="color"
              :class="{ active: selectedColor === color }"
              @click="selectColorForOutside(color)"
              :style="{ backgroundColor: color }"
            ></div>
          </div>
        </div>
        <div class="summary display">
          <h2>Summary</h2>
          <div>
            <p>Color of the laces</p>
            <p class="fontweight">
              {{ selectedLacesColor || "Not selected" }}
            </p>
          </div>
          <div>
            <p>Color of the sole</p>
            <p class="fontweight">
              {{ selectedSoleColor || "Not selected" }}
            </p>
          </div>
          <div>
            <p>Color of the inside of your shoe</p>
            <p class="fontweight">
              {{ selectedInsideColor || "Not selected" }}
            </p>
          </div>
          <div>
            <p>Color of the outside of your shoe</p>
            <p class="fontweight">
              {{ selectedOutsideColor || "Not selected" }}
            </p>
          </div>

          <button @click="submitOrder" class="btn active">Checkout</button>
          <p class="errorMessage"></p>
          <p class="successMessage"></p>
        </div>
        <div class="links">
          <a href="#" class="backButton" style="visibility: hidden">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 320 512"
              style="transform: rotate(180deg)"
            >
              <path
                d="M310.6 233.4c12.5 12.5 12.5 32.8 0 45.3l-192 192c-12.5 12.5-32.8 12.5-45.3 0s-12.5-32.8 0-45.3L242.7 256 73.4 86.6c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0l192 192z"
              ></path>
            </svg>
            <p>Back</p>
          </a>

          <a href="#" class="nextButton" style="visibility: visible">
            <p>Next</p>
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 320 512">
              <path
                d="M310.6 233.4c12.5 12.5 12.5 32.8 0 45.3l-192 192c-12.5 12.5-32.8 12.5-45.3 0s-12.5-32.8 0-45.3L242.7 256 73.4 86.6c-12.5-12.5-12.5-32.8 0-45.3s32.8-12.5 45.3 0l192 192z"
              ></path>
            </svg>
          </a>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  height: 896px;
  width: 100%;
  position: relative;
}

.logo {
  position: fixed;
  top: 24px;
  left: 24px;
  z-index: 10;
  font-size: 1.5rem;
  color: #ffffff;
  text-transform: uppercase;
  font-weight: bold;
  background: rgba(0, 0, 0, 0.5);
  padding: 8px 16px;
  border-radius: 8px;
}

.model {
  width: 100%;
  height: 100%;
  z-index: 0;
  position: relative;
}

.rotate-informer {
  position: absolute;
  top: 55%;
  left: calc(75% / 2 - 12px);
  z-index: 999;
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  border-radius: 4px;
  background-color: #1a1a1a;
  box-shadow: 0 0 8px #0006;
  transition: transform 0.5s;
}

.icons {
  position: absolute;
  top: 48%;
  left: calc(75% / 2 - 12px);
  z-index: 999;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  gap: 24px;
}

.icons .icon {
  display: flex;
  flex-direction: row;
  gap: 8px;
  align-items: center;
}

.icons .icon div {
  border-radius: 100%;
  background-color: #1a1a1a;
  box-shadow: 0 0 8px #0006;
  transition: transform 0.5s;
  width: 40px;
  height: 40px;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
}

.icons .icon p {
  background-color: #1a1a1a;
  color: #fff;
  padding: 8px 12px;
  border-radius: 4px;
  font-size: 14px;
  line-height: 14px;
  margin: 0;
  pointer-events: none;
  transition: opacity 0.4s ease;
  letter-spacing: 1px;
  opacity: 0;
  transition: opacity 0.3s ease;
}

.icons .icon:hover p {
  opacity: 1;
}

.icons div svg {
  width: 24px;
  height: 24px;
  fill: #fff;
}

.rotate-informer p {
  color: var(--white);
}

.config-wrapper {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 24px;
  background: linear-gradient(to bottom, #000000, #473c5d);
  top: 55%;
  right: 0;
  width: 100%;
  height: 45%;
  padding: 16px 52px 24px;
}

.config-wrapper a {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 4px;
}

.config-wrapper a p {
  font-size: 0.8em;
  text-transform: uppercase;
  color: var(--white);
}

.config-wrapper a svg,
li svg {
  text-transform: uppercase;
  color: var(--black);
  width: 10px;
  height: 10px;
  fill: #9b9b9b;
  transform: translateY(-2px);
}

.config-wrapper .links {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  width: 100%;
}

.config-wrapper .elements {
  display: flex;
  flex-direction: column;
  height: 100%;
  justify-content: space-between;
  width: 100%;
}

.config-wrapper .overview,
.config-wrapper .summary {
  display: flex;
  flex-direction: column;
  gap: 16px;
  width: 100%;
}

.config-wrapper .summary {
  display: none;
}

.config-wrapper .summary .fontweight {
  color: var(--white);
}

.config-wrapper h2 {
  text-align: center;
}

.config-wrapper .overview ul li,
.config-wrapper .summary div {
  display: flex;
  width: 100%;
  justify-content: space-between;
  align-items: center;
  padding: 20px 0;
  cursor: pointer;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.colorsItem {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.colorsItem .row {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  gap: 16px;
}

.colorsItem .row div {
  width: 64px;
  height: 64px;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.colorsItem .row div.active {
  border: 3px solid var(--purple);
}

.colorsItem .row div p {
  display: none;
}

.config-wrapper .display {
  display: none;
}

.config-wrapper .visibility {
  visibility: hidden;
}

.btn {
  color: var(--white);
  margin-top: 1rem;
}

.errorMessage {
  color: red;
}

.successMessage {
  color: green;
}

@media (min-width: 1200px) {
  .container {
    flex-direction: row;
    height: 100vh;
    width: 100%;
    justify-content: space-between;
  }

  .model {
    width: 75%;
    height: 100%;
  }

  .icons {
    top: 50%;
    left: 24px;
    flex-direction: column;
    align-items: flex-start;
  }

  .rotate-informer {
    top: 94%;
  }

  .config-wrapper {
    padding: 48px;
    gap: 48px;
    top: 0;
    width: 25%;
    height: 100%;
    right: 0;
    position: absolute;
    overflow-y: auto;
    backdrop-filter: blur(4px);
    -webkit-backdrop-filter: blur(4px);
  }

  .config-wrapper h2 {
    text-align: left;
  }
}
</style>
