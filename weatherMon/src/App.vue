<script setup>
import { ref, watch, onMounted } from 'vue'
import 'leaflet/dist/leaflet.css'
import L from 'leaflet'

const ships = [
  {
    id: 1,
    name: 'Rijeka',
    lat: 45.3271,
    lon: 14.4422,
    status: 'Active',
    engineTemp: 84,
    fuel: 72,
    speed: 28,
    destination: 'Venice',
  },
  {
    id: 2,
    name: 'Lotus',
    lat: 34.0217,
    lon: 9.9278,
    status: 'Maintenance',
    engineTemp: 96,
    fuel: 51,
    speed: 0,
    destination: 'Split Port',
  },
  {
    id: 3,
    name: 'Ithica',
    lat: 38.3643,
    lon: 20.7195,
    status: 'Docked',
    engineTemp: 102,
    fuel: 14,
    speed: 12,
    destination: 'Ithica',
  },
  {
    id: 4,
    name: 'Siren',
    lat: 40.9092,
    lon: 8.8417,
    status: 'Emergency',
    engineTemp: 102,
    fuel: 55,
    speed: 0,
    destination: 'Sicily',
  },
  {
    id: 5,
    name: 'Troy',
    lat: 39.9970,
    lon: 26.1928,
    status: 'Maintenance',
    engineTemp: 61,
    fuel: 55,
    speed: 0,
    destination: 'Hissarlik',
  },
  {
    id: 6,
    name: 'Odysseus',
    lat: 38.9960,
    lon: 18.7482,
    status: 'Active',
    engineTemp: 61,
    fuel: 55,
    speed: 0,
    destination: 'Ithica',
  },
]

const selectedShip = ref(null)
const weather = ref(null)
const activeFilter = ref('All')
const mapMarkers = ref([])

const alerts = ref([
  {
    type: 'Storm Warning',
    message: 'Heavy storm detected near Adriatic route.',
    severity: 'High',
  },
  {
    type: 'Engine Alert',
    message: 'Siren exceeded safe engine temperature.',
    severity: 'Critical',
  },
  {
    type: 'Fuel Spike',
    message: 'Lotus fuel consumption increased by 34%.',
    severity: 'Medium',
  },
])

watch(selectedShip, async (ship) => {
  if (ship) {
    weather.value = await getWeather(ship.lat, ship.lon)
  }
})

async function getWeather(lat, lon) {
  const res = await fetch(
    `https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=wind_speed_10m,precipitation,temperature_2m,relative_humidity_2m`
  )

  const data = await res.json()
  return data.current
}
/*
function getShipStatus(weather) {
  const wind = weather.wind_speed_10m
  const rain = weather.precipitation

  if (wind > 15 || rain > 10) return '#ef4444'
  if (wind > 8 || rain > 3) return '#f59e0b'
  return '#22c55e'
}
*/
function statusColor(status) {
  switch (status) {
    case 'Active':
      return '#4CAF50'
    case 'Maintenance':
      return '#FF9800'
    case 'Emergency':
      return '#F44336'
    case 'Docked':
      return '#FFEB3B'
    default:
      return 'grey'
  }
}

async function renderShips(map) {
  for (const marker of mapMarkers.value) {
    map.removeLayer(marker)
  }

  mapMarkers.value = []

  const filteredShips =
    activeFilter.value === 'All'
      ? ships
      : ships.filter((ship) => ship.status === activeFilter.value)

  for (const ship of filteredShips) {
    const weatherData = await getWeather(ship.lat, ship.lon)
    const color = statusColor(ship.status)

    const marker = L.circleMarker([ship.lat, ship.lon], {
      radius: 10,
      fillColor: color,
      color: '#ffffff',
      weight: 2,
      opacity: 1,
      fillOpacity: 0.9,
    })
      .addTo(map)
      .bindPopup(`
        <div style="font-family: sans-serif; padding: 4px;">
          <h3>${ship.name}</h3>
          <p>Status: ${ship.status}</p>
          <p>Fuel: ${ship.fuel}%</p>
          <p>Engine Temp: ${ship.engineTemp}°C</p>
        </div>
      `)

    marker.on('click', async () => {
      selectedShip.value = ship
      weather.value = weatherData
    })

    mapMarkers.value.push(marker)
  }
}

onMounted(async () => {
  const map = L.map('map').setView([43.5, 15.5], 6)

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '&copy; OpenStreetMap contributors',
  }).addTo(map)

  await renderShips(map)

  watch(activeFilter, async () => {
    await renderShips(map)
  })
})
</script>

<template>
    <v-app>
    <div class="dashboard-wrapper">
      <div class="dashboard-title">
        Fleet Monitoring Dashboard
      </div>

      <!-- KPI CARDS -->
      <v-container fluid>
        <v-row>
          <v-col cols="12" md="3">
            <v-card class="dashboard-card stat-card">
              <div class="stat-label">Active Vessels</div>
              <div class="stat-value">18</div>
              <div class="stat-change green-text">+4% this week</div>
            </v-card>
          </v-col>

          <v-col cols="12" md="3">
            <v-card class="dashboard-card stat-card">
              <div class="stat-label">Incidents</div>
              <div class="stat-value">03</div>
              <div class="stat-change red-text">2 critical alerts</div>
            </v-card>
          </v-col>

          <v-col cols="12" md="3">
            <v-card class="dashboard-card stat-card">
              <div class="stat-label">Fuel Usage</div>
              <div class="stat-value">18.2k L</div>
              <div class="stat-change orange-text">+8% fluctuation</div>
            </v-card>
          </v-col>

          <v-col cols="12" md="3">
            <v-card class="dashboard-card stat-card">
              <div class="stat-label">AI Prediction</div>
              <div class="stat-value">78%</div>
              <div class="stat-change blue-text">Failure probability</div>
            </v-card>
          </v-col>
        </v-row>
      </v-container>

      <v-container fluid>
        <v-row>
          <!-- MAP -->
          <v-col cols="12" lg="8">
            <v-card class="dashboard-card pa-4">
              <div class="section-header">
                <div>
                  <h2>Fleet Locations</h2>
                  <p>Real-time vessel monitoring</p>
                </div>

                <div class="filter-buttons">
                  <v-btn
                    v-for="filter in ['All', 'Active', 'Docked', 'Maintenance', 'Emergency']"
                    :key="filter"
                    rounded="xl"
                    variant="outlined"
                    class="filter-btn"
                    @click="activeFilter = filter"
                  >
                    {{ filter }}
                  </v-btn>
                </div>
              </div>

              <div id="map"></div>
            </v-card>
          </v-col>

          <!-- ALERTS -->
          <v-col cols="12" lg="4">
            <v-card class="dashboard-card pa-4 mb-4">
              <div class="alerts-header-row">
                <div>
                  <h2>Alerts & Incidents</h2>
                  <p>Live system notifications</p>
                </div>

                <v-menu
                  location="bottom end"
                  width="420"
                  content-class="alerts-menu-content"
                >
                  <template #activator="{ props }">
                    <v-btn
                      v-bind="props"
                      color="red"
                      variant="tonal"
                      rounded="xl"
                    >
                      {{ alerts.length }} Active Alerts
                    </v-btn>
                  </template>

                  <v-card class="alerts-dropdown-menu">
                    <v-card-title class="dropdown-menu-title">
                      Active Alerts & Incidents
                    </v-card-title>

                    <v-divider />

                    <v-card-text>
                      <div
                        v-for="alert in alerts"
                        :key="alert.type"
                        class="menu-alert-item"
                      >
                        <div class="menu-alert-top-row">
                          <h3>{{ alert.type }}</h3>

                          <v-chip
                            :color="alert.severity === 'Critical' ? 'red' : 'orange'"
                            variant="tonal"
                            size="small"
                          >
                            {{ alert.severity }}
                          </v-chip>
                        </div>

                        <p class="menu-alert-message">
                          {{ alert.message }}
                        </p>

                        <div class="menu-alert-details">
                          <div class="incident-row">
                            <span>Status: </span>
                            <span class="warning-text">
                              Requires Attention
                            </span>
                          </div>

                          <div class="incident-row">
                            <span>Reported: </span>
                            <span>2 min ago</span>
                          </div>
                        </div>
                      </div>
                    </v-card-text>
                  </v-card>
                </v-menu>
              </div>
            </v-card>

            <!-- SELECTED SHIP -->
            <v-card class="dashboard-card pa-4">
              <div class="section-header-small">
                <h2>Ship Analytics</h2>
                <p>Weather & diagnostics</p>
              </div>

              <v-select
                clearable
                v-model="selectedShip"
                :items="ships"
                item-title="name"
                return-object
                label="Select ship"
                variant="outlined"
                class="mt-4"
              />

              <div v-if="selectedShip" class="ship-info">
                <div class="ship-title-row">
                  <div>
                    <h2>{{ selectedShip.name }}</h2>
                    <p>{{ selectedShip.destination }}</p>
                  </div>

                  <v-chip
                    :color="statusColor(selectedShip.status)"
                    variant="tonal"
                  >
                    {{ selectedShip.status }}
                  </v-chip>
                </div>

                <v-row class="mt-2">
                  <v-col cols="6">
                    <v-card class="metric-card">
                      <img
                        width="22"
                        src="https://img.icons8.com/?size=100&id=DyH5QCwpytAO&format=png&color=FFFFFF"
                      />

                      <div class="metric-label">Wind</div>
                      <div class="metric-value">
                        {{ weather?.wind_speed_10m }} km/h
                      </div>
                    </v-card>
                  </v-col>

                  <v-col cols="6">
                    <v-card class="metric-card">
                      <img
                        width="22"
                        src="https://img.icons8.com/?size=100&id=uMEbUTw18q3z&format=png&color=FFFFFF"
                      />

                      <div class="metric-label">Rain</div>
                      <div class="metric-value">
                        {{ weather?.precipitation }} mm
                      </div>
                    </v-card>
                  </v-col>
                </v-row>

                <v-row>
                  <v-col cols="6">
                    <v-card class="metric-card">
                      <img
                        width="22"
                        src="https://img.icons8.com/?size=100&id=DsREHQ4w4Toq&format=png&color=FFFFFF"
                      />

                      <div class="metric-label">Temperature</div>
                      <div class="metric-value">
                        {{ weather?.temperature_2m }} °C
                      </div>
                    </v-card>
                  </v-col>

                  <v-col cols="6">
                    <v-card class="metric-card">
                      <img
                        width="22"
                        src="https://img.icons8.com/?size=100&id=WlBsGGiFuhgC&format=png&color=FFFFFF"
                      />

                      <div class="metric-label">Humidity</div>
                      <div class="metric-value">
                        {{ weather?.relative_humidity_2m }}%
                      </div>
                    </v-card>
                  </v-col>
                </v-row>
              </div>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </div>
  </v-app>
</template>

<style scoped>
:global(body) {
  background: #0b1120;
  color: white;
  font-family: Inter, sans-serif;
}

.dashboard-wrapper {
  padding: 20px;
}

.dashboard-title {
  font-size: 42px;
  font-weight: 700;
  margin-bottom: 24px;
  color: white;
}

.dashboard-card {
  background: rgba(255, 255, 255, 0.05) !important;
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(18px);
  border-radius: 24px !important;
  color: white;
}

.stat-card {
  padding: 24px;
}

.stat-label {
  color: #94a3b8;
}

.stat-value {
  font-size: 42px;
  font-weight: bold;
  margin-top: 10px;
}

.stat-change {
  margin-top: 12px;
}

.green-text {
  color: #4ade80;
}

.red-text {
  color: #f87171;
}

.orange-text {
  color: #fbbf24;
}

.blue-text {
  color: #38bdf8;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  gap: 20px;
  flex-wrap: wrap;
}

.section-header h2,
.section-header-small h2 {
  font-size: 24px;
}

.section-header p,
.section-header-small p {
  color: #94a3b8;
  margin-top: 4px;
}

.filter-buttons {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.filter-btn {
  border-color: rgba(255,255,255,0.2);
}

#map {
  height: 700px;
  border-radius: 24px;
  overflow: hidden;
  transition: 0.3s;
}

#map:hover {
  box-shadow: 0 0 20px rgba(59, 130, 246, 0.4);
}

.alert-card {
  display: flex;
  justify-content: space-between;
  gap: 16px;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.06);
  border-radius: 18px;
  padding: 16px;
  margin-top: 16px;
}

.alert-card p {
  color: #94a3b8;
  margin-top: 4px;
}

.ship-info {
  margin-top: 20px;
}

.ship-title-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.ship-title-row p {
  color: #94a3b8;
}

.metric-card {
  background: rgba(255,255,255,0.04) !important;
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 20px !important;
  padding: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  color: white;
}

.metric-label {
  margin-top: 10px;
  color: #94a3b8;
}

.metric-value {
  margin-top: 8px;
  font-size: 18px;
  font-weight: bold;
}
</style>
