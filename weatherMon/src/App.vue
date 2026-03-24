<script setup>
import { onMounted } from "vue";
import 'leaflet/dist/leaflet.css';
import L from "leaflet";

onMounted(() => {
  // inicijalizacija mape
  const map = L.map("map").setView([20, 0], 2);

  // OpenStreetMap layer
  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution: "&copy; OpenStreetMap contributors"
  }).addTo(map);

  // test marker (Zagreb)
  L.marker([45.815, 15.981])
    .addTo(map)
    .bindPopup("Zagreb 🚢")
    .openPopup();
});
</script>

<template>
  <h1>Weather Monitoring for Ships</h1>

  <GoogleMap
  api-key="KEY"
  style="width: 100%; height: 500px"
  :center="center"
  :zoom="15"
  >
    <Marker :options="{ position: center }" />
  </GoogleMap>

</template>

<style scoped>

</style>
