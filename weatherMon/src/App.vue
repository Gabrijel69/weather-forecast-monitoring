<script setup>
import {ref, watch, onMounted} from "vue";
import 'leaflet/dist/leaflet.css';
import L from "leaflet";

const ships = ref([]);
let map;

const selectedShip = ref(null);
const weather = ref(null);
const markers = {};

watch(selectedShip, async (ship) => {
  if (ship) {
    weather.value = await getWeather(ship.lat, ship.lon);
  }
});

async function getWeather(lat, lon) {
  const res = await fetch(
    `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=wind_speed_10m,precipitation,temperature_2m,relative_humidity_2m`
  );
  const data = await res.json();
  return data.current;
}

function getShipStatus(weather) {
  const wind = weather.wind_speed_10m;
  const rain = weather.precipitation;
  const temperature = weather.temperature_2m;
  const humidity = weather.relative_humidity_2m;


  if (wind > 15 || rain > 10) return "red";
  if (wind > 8 || rain > 3) return "orange";
  return "green";
}

function addOrUpdateShip(mmsi, lat, lon) {

  let ship = ships.value.find(s => s.mmsi === mmsi);

  if (!ship) {
    ship = { mmsi, name: `Ship ${mmsi}`, lat, lon };
    ships.value.push(ship);
  } else {
    ship.lat = lat;
    ship.lon = lon;
  }

  if (markers[mmsi]) {
    markers[mmsi].setLatLng([lat, lon]);
  } else {

    const marker = L.circleMarker([lat, lon], {
      radius: 5,
      fillColor: "green",
      color: "#0000",
      weight: 5,
      opacity: 1,
      fillOpacity: 0.8
    }).addTo(map);

    marker.bindPopup(`Ship MMSI: ${mmsi}`);

    markers[mmsi] = marker;
  }
}

function connectAISStream() {

  const socket = new WebSocket("wss://stream.aisstream.io/v0/stream");

  socket.onopen = () => {
    console.log("AIS WebSocket connected");

    const subscriptionMessage = {
      APIKey: "API",
      BoundingBoxes: [[[30, -10], [46, 36]]]
    };

    socket.send(JSON.stringify(subscriptionMessage));
  };

  socket.onmessage = async (event) => {

    const text = await event.data.text();   // convert Blob to string
    console.log("AIS data received:", event.data);

    const data = JSON.parse(text);

    if (!data.Message || !data.Message.PositionReport) return;

    const report = data.Message.PositionReport;

    const lat = report.Latitude;
    const lon = report.Longitude;
    const mmsi = report.UserID;

    addOrUpdateShip(mmsi, lat, lon);
  };
}

onMounted(() => {

  map = L.map("map").setView([20, 0], 2);

  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png")
    .addTo(map);

  connectAISStream();

});

</script>

<template>

  <div class="title">Weather Monitoring for Ships</div>

  <v-container>
    <v-row>
      <v-col cols="8">
        <v-sheet class="pa-2">
          <div id="map"></div>
        </v-sheet>
      </v-col>

      <v-col cols="4">
        <v-sheet class="pa-2">
          <v-select
          clearable
          v-model="selectedShip"
          :items="ships"
          item-title="name"
          item-value="name"
          label="Select ship:"
          return-object
          color="#A663CC" outlined dense
          >
            </v-select>
          <div v-if="selectedShip">
            <h3>{{ selectedShip.name }}</h3>
            <p>Latitude: {{ selectedShip.lat }}</p>
            <p>Longitude: {{ selectedShip.lon }}</p>
          </div>
          <v-row>
            <v-col cols="6" class="border-md rounded mx-auto b-color" v-if="selectedShip && weather">
              <v-sheet :elevation="4" class="pa-2 d-flex flex-column align-center">
                <img alt="Wind" loading="lazy" width="20" height="20" decoding="async" src="https://img.icons8.com/?size=100&id=DyH5QCwpytAO&format=png&color=000000">
                <div class="text-center pt-2">
                  Wind: {{ weather.wind_speed_10m }} km/h
                </div>
              </v-sheet>
            </v-col>
            
            <v-col cols="6" class="border-md rounded mx-auto b-color" v-if="selectedShip && weather">
              <v-sheet :elevation="4" class="pa-2 d-flex flex-column align-center">
                <img alt="Precipitation" loading="lazy" width="20" height="20" decoding="async" src="https://img.icons8.com/?size=100&id=uMEbUTw18q3z&format=png&color=000000">
                <div class="text-center pt-2">
                  Rain: {{ weather.precipitation }} mm
                </div>
              </v-sheet>
            </v-col>
          </v-row>
          
          <v-row> 
            <v-col cols="6" class="border-md rounded mx-auto b-color" v-if="selectedShip && weather">
              <v-sheet :elevation="4" class="pa-2 d-flex flex-column align-center">
                <img alt="Temperature" loading="lazy" width="20" height="20" decoding="async" src="https://img.icons8.com/?size=100&id=DsREHQ4w4Toq&format=png&color=000000">
                <div class="text-center pt-2">
                  Temperature: {{ weather.temperature_2m}} °C
                </div>
              </v-sheet>
            </v-col>

            <v-col cols="6" class="border-md rounded mx-auto b-color" v-if="selectedShip && weather">
              <v-sheet :elevation="4" class="pa-2 d-flex flex-column align-center">
                <img alt="Humidity" loading="lazy" width="20" height="20" decoding="async" src="https://img.icons8.com/?size=100&id=WlBsGGiFuhgC&format=png&color=000000">
                <div class="text-center pt-2">
                  Humidity: {{ weather.relative_humidity_2m }} %
                </div>
              </v-sheet>
            </v-col>
          </v-row>
          
        </v-sheet>
      </v-col>
    </v-row>
  </v-container>
</template>

<style scoped>
:global(body) {
  background-color: #1f1f1f;
  padding: 10px 30px 10px;
}

.title {
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 40px;
  font-weight: bold;
  color: #6F2DBD;
}

#map{
  height: 800px;
  border-radius: 25px;
  transition: 0.4s;
}

select {
  border:1px solid var(--vt-c-divider);
  background-color: #A663CC;
  border-radius:4px;
  margin-top:10px;
  padding:.2em .6em;
  transition:background-color .5s
}

.dropdown-text-color{
  color: #E0E0E0;
  margin: 3px;
}

#map:hover{
  box-shadow: 0 0 10px 5px #6F2DBD;
}

.b-color{
  border-color: #A663CC;
}
</style>
