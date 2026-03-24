<script setup>
import { onMounted } from "vue";
import 'leaflet/dist/leaflet.css';
import L from "leaflet";

onMounted(() => {
  // inicijalizacija mape
  const map = L.map("map").setView([45.815, 15.981], 5);

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

  <div id="map" style="height: 800px;"></div>

</template>

<style scoped>

</style>
