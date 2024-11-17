<script lang="ts" setup>
import { ref, watch, onMounted } from "vue";
import { useRoute } from "vue-router";
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";

const route = useRoute();
const productCode = ref<string | undefined>(undefined);
const productData = ref<{ productName: string; productPrice: number } | null>(
  null
);
const isLoading = ref<boolean>(true);
const error = ref<string | null>(null);
const showBackButton = ref<boolean>(false);
const showNextButton = ref<boolean>(true);

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
  } catch (err) {
    console.error("Error occurred:", err);
    error.value = "Unable to fetch product information.";
  } finally {
    isLoading.value = false;
  }
}

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
      gltf.scene.scale.set(50, 50, 50);
      gltf.scene.position.set(0, 0, 0);
      scene.add(gltf.scene);

      const controls = new OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
      controls.dampingFactor = 0.25;
      controls.enableZoom = true;
      controls.target.set(0, 0, 0);

      controls.update();
      animate();
    },
    undefined,
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
</script>

<template>
  <div class="container">
    <h3 class="logo">REBILT</h3>
    <div class="model"></div>
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
          </ul>
        </div>
        <div class="config-ui__page page1 colorsItem display">
          <h2>Choose the color of the laces</h2>
          <div class="row">
            <div style="background-color: #000">
              <p>#000</p>
            </div>
            <div style="background-color: #000">
              <p>#000</p>
            </div>
            <div style="background-color: #000">
              <p>#000</p>
            </div>
            <div style="background-color: #000">
              <p>#000</p>
            </div>
            <div style="background-color: #000">
              <p>#000</p>
            </div>
          </div>
        </div>
        <div class="config-ui__page page2 colorsItem display">
          <h2>Choose the color of the sole</h2>
          <div class="row">
            <div style="background-color: #000">
              <p>#000</p>
            </div>
            <div style="background-color: #000">
              <p>#000</p>
            </div>
            <div style="background-color: #000">
              <p>#000</p>
            </div>
            <div style="background-color: #000">
              <p>#000</p>
            </div>
            <div style="background-color: #000">
              <p>#000</p>
            </div>
          </div>
        </div>
        <div class="summary display">
          <h2>Summary</h2>
          <div>
            <p>Color of the laces</p>
            <p class="fontWeight"></p>
          </div>
          <div>
            <p>Color of the sole</p>
            <p class="fontWeight"></p>
          </div>
          <button class="btn active">Checkout</button>
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
}
</style>
