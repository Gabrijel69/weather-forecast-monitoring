<script setup>
import {ref} from "vue";
import { onMounted } from "vue";
import 'leaflet/dist/leaflet.css';
import L from "leaflet";

const ships = [
    { name: "Ship 1", lat: 34.10, lon: -41.19 },
    { name: "Ship 2", lat: 45.34, lon: 14.40}
  ];

async function getWeather(lat, lon) {
  const res = await fetch(
    `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=wind_speed_10m,precipitation`
  );
  const data = await res.json();
  return data.current;t
}

function getShipStatus(weather) {
  const wind = weather.wind_speed_10m;
  const rain = weather.precipitation;

  if (wind > 15 || rain > 10) return "red";
  if (wind > 8 || rain > 3) return "orange";
  return "green";
}

onMounted(async () => {
  const map = L.map("map").setView([20, 0], 2);

  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png")
    .addTo(map);

  for (const ship of ships) {
    const weather = await getWeather(ship.lat, ship.lon);
    const status = getShipStatus(weather);

    const myIcon = {
      radius: 5,
      fillColor: status,
      color: "#0000",
      weight: 5,
      opacity: 1,
      fillOpacity: 0.8
    };


    L.circleMarker([ship.lat, ship.lon], myIcon).addTo(map)
    .bindPopup(`
        <b>${ship.name}</b><br>
        Wind: ${weather.wind_speed_10m} km/h<br>
        Rain: ${weather.precipitation} mm
      `);
  }
});

const selectedShip = ref(null);

</script>

<template>

  <div class="title"><h1>Weather Monitoring for Ships</h1></div>

  <div class="dropdown-align">
    <div class="dropdown">

      <div>Odaberi brod:</div>

      <select v-model="selectedShip">
        <option v-for="ship in ships" :value="ship">
          {{ ship.name }}
        </option>
      </select>
        <div v-if="selectedShip">
          <h3>{{ selectedShip.name }}</h3>
          <p>Latitude: {{ selectedShip.lat }}</p>
          <p>Longitude: {{ selectedShip.lon }}</p>
        </div>
    </div>
  
  </div>

  <div id="map" style="height: 800px;"></div>

</template>

<style scoped>
:global(body) {
  background-color: #1f1f1f;
  color: #8140B3;
  padding: 10px 30px 10px;
}

.title {
  display: flex;
  justify-content: center;
  align-items: center;
}

.dropdown-align{
  display: flex;
  justify-content: flex-end;
}

.dropdown{
  display: flex;
  flex-direction: column;
}

#map{
  margin-top: 5px;
  margin-bottom: 5px;
  border-radius: 25px;
}
</style>
