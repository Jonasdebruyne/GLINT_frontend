<script lang="ts" setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import Navigation from "../../components/navComponent.vue";

// Basis-URL afhankelijk van de omgeving
const isProduction = window.location.hostname !== "localhost";
const baseURL = isProduction
  ? "https://glint-backend-admin.onrender.com/api/v1"
  : "http://localhost:3000/api/v1";

// Router en route-instellingen
const router = useRouter();
const route = useRoute();
const jwtToken = localStorage.getItem("jwtToken");

if (!jwtToken) {
  router.push("/login");
}

// Verkrijg de productCode uit de routeparameter
const productCode = ref(route.params.id as string);

// Orderdata en andere variabelen
const orderData = ref<any>(null);
const orderStatus = ref<string>("");

const fetchOrderData = async () => {
  if (!productCode.value) {
    console.error("Geen productcode beschikbaar.");
    return;
  }

  try {
    const response = await fetch(`${baseURL}/orders/${productCode.value}`);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    orderData.value = data.data.order;

    if (orderData.value) {
      orderStatus.value = orderData.value.orderStatus;
    }
  } catch (error) {
    console.error("Error fetching order data:", error);
    alert("Er is een fout opgetreden bij het ophalen van de ordergegevens.");
  }
};

onMounted(() => {
  fetchOrderData();
});

// Order bijwerken
const updateOrder = async () => {
  if (!orderStatus.value) {
    alert("Vul alstublieft alle vereiste velden in.");
    return;
  }

  try {
    const response = await fetch(`${baseURL}/orders/${productCode.value}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        orderStatus: orderStatus.value,
        lacesColor: orderData.value.lacesColor, // Vullen met de waarde van de bestaande kleur
        soleColor: orderData.value.soleColor,
        insideColor: orderData.value.insideColor,
        outsideColor: orderData.value.outsideColor,
      }),
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const result = await response.json();
    router.push("/admin/orders"); // Redirect na succesvolle update
  } catch (error) {
    console.error("Error updating order:", error);
    alert("Er is een fout opgetreden bij het bijwerken van de order.");
  }
};
</script>

<template>
  <Navigation />
  <div class="content">
    <h1>Order Bewerken</h1>
    <form v-if="orderData" @submit.prevent="updateOrder">
      <div class="row">
        <div class="column">
          <label for="orderStatus">Status:</label>
          <select v-model="orderStatus" id="orderStatus" required>
            <option value="pending">Pending</option>
            <option value="shipped">Shipped</option>
            <option value="delivered">Delivered</option>
            <option value="cancelled">Cancelled</option>
          </select>
        </div>
      </div>
      <button type="submit" class="btn active">Bewerk Order</button>
    </form>
  </div>
</template>

<style scoped>
/* Stijlen voor je formulier en pagina */
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
