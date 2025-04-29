<script setup>
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
