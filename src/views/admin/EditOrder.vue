<script lang="ts" setup>
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
const orderId = ref<string>("");

const orderData = ref<any>(null);
const shippingAddress = ref<string>("");
const orderStatus = ref<string>("");
const totalPrice = ref<number | null>(null);

const fetchOrderData = async () => {
  const id = route.params.id; // Haal order-ID op uit de routeparameters
  orderId.value = id;

  try {
    const response = await fetch(`${baseURL}/orders/${id}`);
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    const data = await response.json();
    orderData.value = data.data.order;

    if (orderData.value) {
      // Vul de ordergegevens in de variabelen
      shippingAddress.value = orderData.value.shippingAddress.city;
      orderStatus.value = orderData.value.orderStatus;
      totalPrice.value = orderData.value.totalPrice;
    }
  } catch (error) {
    console.error("Error fetching order data:", error);
    alert("Er is een fout opgetreden bij het ophalen van de ordergegevens.");
  }
};

onMounted(() => {
  fetchOrderData();
});

const updateOrder = async () => {
  if (
    !orderStatus.value ||
    !shippingAddress.value ||
    totalPrice.value === null
  ) {
    alert("Vul alstublieft alle vereiste velden in.");
    return;
  }

  try {
    const response = await fetch(`${baseURL}/orders/${orderId.value}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        shippingAddress: {
          city: shippingAddress.value, // Controleer of je het volledige adres nodig hebt of alleen de stad.
        },
        orderStatus: orderStatus.value,
        totalPrice: totalPrice.value,
      }),
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const result = await response.json();
    router.push("/admin/orders");
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
          <label for="shippingAddress">Verzendadres:</label>
          <input
            v-model="shippingAddress"
            id="shippingAddress"
            type="text"
            required
          />
        </div>
        <div class="column">
          <label for="orderStatus">Status:</label>
          <select v-model="orderStatus" id="orderStatus">
            <option value="pending">Pending</option>
            <option value="shipped">Shipped</option>
            <option value="delivered">Delivered</option>
            <option value="cancelled">Cancelled</option>
          </select>
        </div>
      </div>
      <div class="column">
        <label for="totalPrice">Totaalprijs:</label>
        <input v-model="totalPrice" id="totalPrice" type="number" required />
      </div>
      <button type="submit" class="btn active">Bewerk Order</button>
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
