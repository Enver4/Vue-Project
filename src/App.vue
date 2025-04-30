<!-- <script setup>
import { ref, onMounted } from "vue";


const exchangeRates = ref([]);
const error = ref(null);

onMounted(async () => {
  try {
    const response = await fetch("http://localhost:5089/api/FPMS");
    if (!response.ok) throw new Error(`API error: ${response.status}`);
    exchangeRates.value = await response.json();
  } catch (err) {
    error.value = err.message;
  }
});
</script>

<template>
  <div>
    <h1>All Exchange Rates</h1>
    <div v-if="error">Error: {{ error }}</div>
    <ul v-else>
      <li v-for="rate in exchangeRates" :key="rate.id">
        {{ rate.fromCurrency }} → {{ rate.toCurrency }} = {{ rate.rate }} ({{
          rate.updatedAt
        }})
      </li>
    </ul>
  </div>
 
</template>

<style >
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
li {
  border: 1px solid grey;
  padding: 10px;
  color: black;
  font-weight: bolder;
}
h1 {
  color: green;
  margin-bottom: 5px;
}
</style>
 -->

<template>
  <div class="p-4">
    <h1 class="text-xl font-bold mb-2">Currency API Tester</h1>

    <button @click="callFPMS" class="btn">Call FPMS</button>
    <button @click="callLatestRates" class="btn ml-2">Call Latest Rates</button>
    <!-- <button @click="addLatestRates" class="btn">Add Latest Rates</button> -->
    <div class="mt-4">
      <input
        v-model="inputText"
        placeholder="Enter input like: 100 * USD"
        class="border px-2 py-1"
      />
      <button @click="calculate" class="btn ml-2">Calculate</button>
    </div>

    <div class="mt-4 text-green-700 font-mono">
      <div v-if="error">Error: {{ error }}</div>
      <ul v-else>
        <li v-for="rate in response" :key="rate.id">
          {{ rate.fromCurrency }} → {{ rate.toCurrency }} = {{ rate.rate }} ({{
            formatDate(rate.updatedAt)
          }})
        </li>
      </ul>
      <div><strong>Response:</strong> {{ calcAnswer }}</div>
    </div>

    <div>
      <button @click="showAuthModal = true" class="btn-update">
        Update Exchange Rates
      </button>

      <div v-if="showAuthModal" class="modal-overlay">
        <div class="modal">
          <h2>Admin Login</h2>
          <input v-model="auth.username" placeholder="Username" class="input" />
          <input
            v-model="auth.password"
            placeholder="Password"
            type="password"
            class="input"
          />
          <div class="actions">
            <button @click="authenticateAndUpdateRates" class="btn">
              Submit
            </button>
            <button @click="showAuthModal = false" class="btn btn-cancel">
              Cancel
            </button>
          </div>
          <p v-if="authError" class="error">{{ authError }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
const showAuthModal = ref(false);
const auth = ref({ username: "", password: "" });
const authError = ref("");

const API_BASE = "http://localhost:5089/api/FPMS";

const response = ref("");
const inputText = ref("");
const calcAnswer = ref("");
const callFPMS = async () => {
  try {
    const res = await fetch(`${API_BASE}`);
    response.value = await res.json();
  } catch (err) {
    response.value = "Error: " + err.message;
  }
};

const callLatestRates = async () => {
  try {
    const res = await fetch(`${API_BASE}/Latest Rates`);
    response.value = await res.json();
  } catch (err) {
    response.value = "Error: " + err.message;
  }
};

const calculate = async () => {
  try {
    const encodedInput = encodeURIComponent(inputText.value);
    const res = await fetch(`${API_BASE}/calculate?input=${encodedInput}`);
    calcAnswer.value = await res.text();
  } catch (err) {
    calcAnswer.value = "Error: " + err.message;
  }
};

const formatDate = (isoString) => {
  const date = new Date(isoString);

  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, "0");
  const day = String(date.getDate()).padStart(2, "0");
  const hours = String(date.getHours()).padStart(2, "0");
  const minutes = String(date.getMinutes()).padStart(2, "0");
  const seconds = String(date.getSeconds()).padStart(2, "0");

  return `${year}-${month}-${day} ${hours}:${minutes}:${seconds}`;
};

/* const addLatestRates = async () => {
  try {
    const res = await fetch(`http://localhost:5089/api/ExchangeRates/update-live-rates`)
    response.value = await res.json()
  } catch (err) {
    response.value = 'Error: ' + err.message
  }
} */

const loginEndpoint = "http://localhost:5089/api/auth/login";

const authenticateAndUpdateRates = async () => {
  try {
    // Step 1: Login and get JWT token
    const loginResponse = await fetch(loginEndpoint, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        username: auth.value.username,
        password: auth.value.password,
      }),
    });

    if (!loginResponse.ok) {
      throw new Error("Login failed: Invalid credentials");
    }

    const loginData = await loginResponse.json();
    const token = loginData.token;

    if (!token) {
      throw new Error("No token received");
    }

    // Step 2: Call protected update endpoint
    const updateResponse = await fetch(
      "http://localhost:5089/api/ExchangeRates/update-live-rates",
      {
        method: "POST",
        headers: {
          Authorization: `Bearer ${token}`,
        },
      }
    );

    if (!updateResponse.ok) {
      throw new Error("Update failed: Not authorized or server error");
    }

    const result = await updateResponse.text();
    alert(result);
    showAuthModal.value = false;
    authError.value = "";
  } catch (error) {
    authError.value = error.message;
  }
};
</script>
<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal {
  background: white;
  padding: 2rem;
  border-radius: 10px;
  min-width: 300px;
}

.input {
  display: block;
  margin: 0.5rem 0;
  width: 100%;
  padding: 0.5rem;
}

.actions {
  display: flex;
  justify-content: space-between;
  margin-top: 1rem;
}

.btn {
  padding: 2px 10px;
  cursor: pointer;
  margin-right: 5px;
  margin-bottom: 10px;
  margin-left: 5px;
}
.btn:hover {
  background-color: rgba(27, 87, 216, 0.507);
}

.btn-cancel {
  background: #ccc;
}

.error {
  color: red;
  margin-top: 1rem;
}
</style>

<style>
.btn {
  background-color: #2563eb;
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 0.375rem;
  font-weight: 600;
}
.btn:hover {
  background-color: #1d4ed8;
}

.btn-update {
  border-radius: 5px;
  border: 2px solid grey;
  background-color: #2563eb;
  color: white;
}
</style>
