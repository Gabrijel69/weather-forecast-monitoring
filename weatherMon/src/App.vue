<script setup>
import { onMounted } from "vue";
import 'leaflet/dist/leaflet.css';
import L from "leaflet";

async function getWeather(lat, lon) {
  const res = await fetch(
    `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=wind_speed_10m,precipitation`
  );
  const data = await res.json();
  return data.current;
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

  const ships = [
    { name: "Ship 1", lat: 45.815, lon: 15.981 },
    { name: "Ship 2", lat: 40.7, lon: -74 }
  ];

  for (const ship of ships) {
    const weather = await getWeather(ship.lat, ship.lon);
    const status = getShipStatus(weather);

    const icon = L.divIcon({
      html: `<div style="
        background:${status};
        width:15px;
        height:15px;
        border-radius:50%;
      "></div>`
    });

    L.marker([ship.lat, ship.lon], { icon })
      .addTo(map)
      .bindPopup(`
        <b>${ship.name}</b><br>
        Wind: ${weather.wind_speed_10m} km/h<br>
        Rain: ${weather.precipitation} mm
      `);
  }
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
