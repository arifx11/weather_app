  <template>
    <div class="weather-page">
      <v-container class="py-6 pb-12" style="max-width: 1100px">
        <h1 class="text-h5 font-weight-bold mb-4">Weather Forecast</h1>

        <!-- Search -->
        <v-row align="center" class="mb-2">
          <v-col cols="12" md="8">
            <v-autocomplete v-model="selected" v-model:search="query" :items="suggestions" item-title="label"
              return-object no-filter hide-no-data hide-details variant="outlined" density="comfortable"
              placeholder="Search for a city..." prepend-inner-icon="mdi-magnify"
              :menu-props="{ contentClass: 'weather-menu' }" @update:model-value="selectPlace" />
          </v-col>
          <v-col cols="12" md="4" class="d-flex align-center ga-2">
            <v-btn variant="tonal" prepend-icon="mdi-crosshairs-gps" @click="useMyLocation">
              My location
            </v-btn>
            <v-btn variant="tonal" icon="mdi-refresh" :loading="loading" @click="loadWeather" />
          </v-col>
        </v-row>

        <v-alert v-if="error" type="error" variant="tonal" closable class="mb-4">
          {{ error }}
        </v-alert>

        <div v-if="!weather && loading" class="text-center pa-10">
          <v-progress-circular indeterminate size="48" color="primary" />
        </div>

        <template v-if="weather">
          <!-- Current conditions -->
          <v-card class="hero pa-6 mb-4" :class="current.is_day ? 'hero-day' : 'hero-night'" elevation="3">
            <div class="d-flex flex-wrap justify-space-between align-center ga-4">
              <div>
                <div class="text-h6">{{ placeLabel }}</div>
                <div class="text-body-2 opacity-80">{{ today }}</div>
                <div class="hero-temp my-2">{{ Math.round(current.temperature_2m) }}°C</div>
                <div class="text-h6">{{ now.label }}</div>
                <div class="text-body-2 opacity-80">
                  Feels like {{ Math.round(current.apparent_temperature) }}°C
                </div>
              </div>
              <v-icon :icon="now.icon" size="120" />
            </div>
          </v-card>

          <!-- Stats -->
          <v-row class="mb-2">
            <v-col v-for="s in stats" :key="s.label" cols="6" md="3" class="stat-col">
              <v-card variant="outlined" class="stat-card pa-3 text-center h-100">
                <v-icon :icon="s.icon" size="28" />
                <div class="text-subtitle-1 font-weight-bold mt-1">{{ s.value }}</div>
                <div class="text-caption">{{ s.label }}</div>
              </v-card>
            </v-col>
          </v-row>

          <!-- Hourly -->
          <v-card variant="outlined" class="pa-4 mb-4">
            <div class="text-subtitle-1 font-weight-bold mb-3 section-title">Next 12 hours</div>
            <div class="hourly">
              <div v-for="h in hourly" :key="h.time" class="hour">
                <div class="text-caption">{{ hourLabel(h.time) }}</div>
                <v-icon :icon="h.icon" size="28" class="my-2" />
                <div class="font-weight-bold">{{ h.temp }}°</div>
                <div class="text-caption text-primary">{{ h.rain }}%</div>
              </div>
            </div>
          </v-card>

          <!-- 7-day -->
          <v-card variant="outlined" class="pa-4">
            <div class="text-subtitle-1 font-weight-bold mb-2 section-title">3-day forecast</div>
            <div v-for="(d, i) in daily" :key="d.date" class="day-row">
              <div class="day-name">{{ dayLabel(d.date, i) }}</div>
              <v-icon :icon="d.icon" size="24" />
              <div class="text-caption text-primary day-rain">{{ d.rain }}%</div>
              <div class="day-temps">
                <span class="font-weight-bold">{{ d.max }}°</span>
                <span class="text-medium-emphasis"> / {{ d.min }}°</span>
              </div>
            </div>
          </v-card>

          <div class="text-caption text-medium-emphasis text-center mt-4">
            Updated {{ clock(updatedAt) }} · refreshes every 5 minutes · Data by Open-Meteo
          </div>
        </template>
      </v-container>
    </div>
  </template>

<script setup>
const place = ref({
  name: 'San Fernando',
  admin1: 'Pampanga',
  country: 'Philippines',
  latitude: 15.0286,
  longitude: 120.6897,
})
const weather = ref(null)
const loading = ref(false)
const error = ref('')
const updatedAt = ref(null)

const query = ref('')
const selected = ref(null)
const suggestions = ref([])

let timer = null
let searchTimer = null

// Weather code -> label + icon
const codes = {
  0: ['Clear sky', 'mdi-weather-sunny'],
  1: ['Mainly clear', 'mdi-weather-sunny'],
  2: ['Partly cloudy', 'mdi-weather-partly-cloudy'],
  3: ['Overcast', 'mdi-weather-cloudy'],
  45: ['Fog', 'mdi-weather-fog'],
  48: ['Fog', 'mdi-weather-fog'],
  51: ['Light drizzle', 'mdi-weather-rainy'],
  53: ['Drizzle', 'mdi-weather-rainy'],
  55: ['Heavy drizzle', 'mdi-weather-rainy'],
  61: ['Light rain', 'mdi-weather-rainy'],
  63: ['Rain', 'mdi-weather-rainy'],
  65: ['Heavy rain', 'mdi-weather-pouring'],
  80: ['Light showers', 'mdi-weather-rainy'],
  81: ['Showers', 'mdi-weather-pouring'],
  82: ['Heavy showers', 'mdi-weather-pouring'],
  95: ['Thunderstorm', 'mdi-weather-lightning-rainy'],
  96: ['Thunderstorm', 'mdi-weather-lightning-rainy'],
  99: ['Severe thunderstorm', 'mdi-weather-lightning-rainy'],
}

function info(code, isDay = 1) {
  const [label, icon] = codes[code] ?? ['Unknown', 'mdi-weather-cloudy']
  return { label, icon: !isDay && icon === 'mdi-weather-sunny' ? 'mdi-weather-night' : icon }
}


async function loadWeather() {
  loading.value = true
  error.value = ''

  try {
    const response = await $fetch(
      'https://api.weatherapi.com/v1/forecast.json',
      {
        query: {
          key: 'd84648f7e8374e01aea35501251908',
          q: place.value.latitude && place.value.longitude
            ? `${place.value.latitude},${place.value.longitude}`
            : place.value.name,
          days: 3,
          aqi: 'no',
          alerts: 'no'
        }
      }
    )

    // Convert WeatherAPI response to your existing data format
    const conditionCode = (text) => {
      const t = text.toLowerCase()

      if (t.includes('thunder')) return 95
      if (t.includes('snow') || t.includes('ice')) return 71
      if (t.includes('heavy rain')) return 65
      if (t.includes('rain')) return 63
      if (t.includes('drizzle')) return 53
      if (t.includes('fog') || t.includes('mist')) return 45
      if (t.includes('cloud')) return 3
      if (t.includes('clear') || t.includes('sun')) return 0

      return 2
    }

    const current = response.current
    const forecastDays = response.forecast.forecastday

    weather.value = {
      current: {
        time: response.location.localtime,
        temperature_2m: current.temp_c,
        apparent_temperature: current.feelslike_c,
        relative_humidity_2m: current.humidity,
        precipitation: current.precip_mm,
        weather_code: conditionCode(current.condition.text),
        wind_speed_10m: current.wind_kph,
        is_day: current.is_day
      },

      hourly: {
        time: forecastDays.flatMap(d =>
          d.hour.map(h => h.time)
        ),
        temperature_2m: forecastDays.flatMap(d =>
          d.hour.map(h => h.temp_c)
        ),
        weather_code: forecastDays.flatMap(d =>
          d.hour.map(h => conditionCode(h.condition.text))
        ),
        precipitation_probability: forecastDays.flatMap(d =>
          d.hour.map(h => h.chance_of_rain)
        )
      },

      daily: {
        time: forecastDays.map(d => d.date),
        weather_code: forecastDays.map(d =>
          conditionCode(d.day.condition.text)
        ),
        temperature_2m_max: forecastDays.map(d =>
          d.day.maxtemp_c
        ),
        temperature_2m_min: forecastDays.map(d =>
          d.day.mintemp_c
        ),
        precipitation_probability_max: forecastDays.map(d =>
          d.day.daily_chance_of_rain
        ),
        sunrise: forecastDays.map(d =>
          `${d.date} ${d.astro.sunrise}`
        ),
        sunset: forecastDays.map(d =>
          `${d.date} ${d.astro.sunset}`
        )
      }
    }

    // Update location information
    place.value.name = response.location.name
    place.value.admin1 = response.location.region
    place.value.country = response.location.country

    updatedAt.value = new Date()

  } catch (e) {
    error.value =
      e?.data?.error?.message ||
      'Could not load weather data. Check your internet connection and try again.'
  } finally {
    loading.value = false
  }
}

// City search (debounced)
watch(query, (q) => {
  clearTimeout(searchTimer)
  if (!q || q.length < 2 || q === selected.value?.label) return
  searchTimer = setTimeout(async () => {
    try {
      const res = await $fetch('https://geocoding-api.open-meteo.com/v1/search', {
        query: { name: q, count: 6, language: 'en', format: 'json' },
      })
      suggestions.value = (res.results ?? []).map((r) => ({
        ...r,
        label: [r.name, r.admin1, r.country].filter(Boolean).join(', '),
      }))
    } catch (e) {
      suggestions.value = []
    }
  }, 350)
})

function selectPlace(item) {
  if (!item) return
  place.value = item
  loadWeather()
}

function useMyLocation() {
  if (!navigator.geolocation) {
    error.value = 'Your browser does not support location access.'
    return
  }
  navigator.geolocation.getCurrentPosition(
    (pos) => {
      place.value = {
        name: 'My location',
        latitude: pos.coords.latitude,
        longitude: pos.coords.longitude,
      }
      selected.value = null
      loadWeather()
    },
    () => {
      error.value = 'Location access was denied. Search for a city instead.'
    }
  )
}

// Derived data
const current = computed(() => weather.value?.current)
const now = computed(() => (current.value ? info(current.value.weather_code, current.value.is_day) : null))
const placeLabel = computed(() =>
  [place.value.name, place.value.admin1, place.value.country].filter(Boolean).join(', ')
)
const today = computed(() =>
  new Date().toLocaleDateString([], { weekday: 'long', month: 'long', day: 'numeric' })
)

const hourly = computed(() => {
  if (!weather.value) return []
  const h = weather.value.hourly
  const nowKey = weather.value.current.time.slice(0, 13) + ':00'
  const start = Math.max(h.time.findIndex((t) => t >= nowKey), 0)
  return h.time.slice(start, start + 12).map((t, i) => {
    const hr = Number(t.slice(11, 13))
    return {
      time: t,
      temp: Math.round(h.temperature_2m[start + i]),
      rain: h.precipitation_probability[start + i],
      ...info(h.weather_code[start + i], hr >= 6 && hr < 18 ? 1 : 0),
    }
  })
})

const daily = computed(() => {
  if (!weather.value) return []
  const d = weather.value.daily
  return d.time.map((t, i) => ({
    date: t,
    max: Math.round(d.temperature_2m_max[i]),
    min: Math.round(d.temperature_2m_min[i]),
    rain: d.precipitation_probability_max[i],
    ...info(d.weather_code[i]),
  }))
})

const stats = computed(() =>
  current.value
    ? [
      { icon: 'mdi-water-percent', label: 'Humidity', value: `${current.value.relative_humidity_2m}%` },
      { icon: 'mdi-weather-windy', label: 'Wind', value: `${Math.round(current.value.wind_speed_10m)} km/h` },
      { icon: 'mdi-weather-pouring', label: 'Rain chance today', value: `${daily.value[0].rain}%` },
      { icon: 'mdi-weather-sunset', label: 'Sunset', value: clock(weather.value.daily.sunset[0]) },
    ]
    : []
)

// Formatters
const clock = (t) => new Date(t).toLocaleTimeString([], { hour: 'numeric', minute: '2-digit' })
const hourLabel = (t) => new Date(t).toLocaleTimeString([], { hour: 'numeric' })
const dayLabel = (t, i) =>
  i === 0 ? 'Today' : new Date(t + 'T00:00').toLocaleDateString([], { weekday: 'long' })

onMounted(() => {
  loadWeather()
  timer = setInterval(loadWeather, 5 * 60 * 1000) // auto-refresh every 5 minutes
})
onBeforeUnmount(() => clearInterval(timer))
</script>

<!-- Not scoped on purpose: Vuetify's inner elements and the dropdown menu
      live outside this component's scope ID. -->
<style>
/* ---------- Weather page: White and Red Theme ---------- */

.weather-page {
  --v-theme-primary: 198, 40, 40;
  --v-theme-surface: 255, 255, 255;
  --v-theme-on-surface: 0, 0, 0;
  --v-theme-background: 255, 255, 255;
  --v-theme-on-background: 0, 0, 0;
  --v-border-color: 198, 40, 40;
  --v-border-opacity: 0.35;

  min-height: calc(100vh - 64px);
  background: #ffffff;
  color: #000000;
}

/* Page background */
.v-main:has(.weather-page) {
  background: #ffffff;
}

/* Title */
.weather-page h1 {
  color: #000000;
}

/* Search field */
.weather-page .v-field {
  background: #fff5f5;
  color: #000000;
  border-radius: 10px;
}

.weather-page .v-field--variant-outlined .v-field__outline {
  color: #c62828;
}

.weather-page .v-field .v-icon {
  color: #c62828;
}

.weather-page input {
  color: #000000;
}

.weather-page input::placeholder {
  color: #777777;
  opacity: 1;
}

/* Buttons */
.weather-page .v-btn--variant-tonal {
  background: #ffebee;
  color: #b71c1c;
  border: 1px solid #ef9a9a;
  height: 48px !important;
  border-radius: 10px;
}

.weather-page .v-btn--icon.v-btn--variant-tonal {
  width: 48px !important;
}

.weather-page .v-btn:hover {
  background: #ffcdd2;
}

/* Loading indicator */
.weather-page .v-progress-circular {
  color: #c62828 !important;
}

/* Hero card */
.weather-page .hero {
  color: #000000;
  border: 1px solid #ef9a9a;
  border-radius: 14px;
  box-shadow: 0 5px 20px rgba(198, 40, 40, 0.12);
}

.weather-page .hero-day {
  background: linear-gradient(115deg, #ffcdd2, #ffebee, #ef9a9a);
}

.weather-page .hero-night {
  background: linear-gradient(115deg, #f8bbd0, #fce4ec, #ffcdd2);
}

.weather-page .hero .v-icon {
  color: #b71c1c;
}

.hero-temp {
  font-size: 80px;
  font-weight: 300;
  line-height: 1.1;
  color: #000000;
}

/* Outlined cards */
.weather-page .v-card--variant-outlined {
  background: #fff5f5;
  color: #000000;
  border: 1px solid #ef9a9a;
  border-radius: 12px;
}

/* Card icons */
.weather-page .v-card--variant-outlined .v-icon {
  color: #c62828;
}

/* Section titles */
.weather-page .section-title {
  color: #b71c1c;
}

/* Primary text */
.weather-page .text-primary {
  color: #c62828 !important;
}

/* Hourly forecast */
.hourly {
  display: flex;
  overflow-x: auto;
}

.hour {
  flex: 1 0 64px;
  text-align: center;
  padding: 0 6px;
  border-right: 1px solid #ffcdd2;
}

.hour:last-child {
  border-right: none;
}

/* 3-day forecast rows */
.day-row {
  display: grid;
  grid-template-columns: 1fr 40px 60px 90px;
  align-items: center;
  gap: 10px;
  padding: 12px 4px;
  border-top: 1px solid #ffcdd2;
}

.day-name {
  color: #000000;
}

.day-rain,
.day-temps {
  text-align: right;
}

/* Autocomplete dropdown */
.weather-menu .v-list {
  background: #ffffff !important;
  color: #000000 !important;
  border: 1px solid #ef9a9a;
  border-radius: 10px;
}

.weather-menu .v-list-item {
  color: #000000 !important;
}

.weather-menu .v-list-item:hover {
  background: #ffebee !important;
}

/* Stat card hover animations */
.weather-page .v-card.stat-card {
  cursor: pointer;
  transition:
    transform 0.25s ease,
    box-shadow 0.25s ease,
    border-color 0.25s ease,
    background 0.25s ease;
}

.weather-page .v-card.stat-card:hover {
  transform: translateY(-6px);
  background: #ffebee;
  border-color: #c62828;
  box-shadow: 0 10px 24px rgba(198, 40, 40, 0.18);
}

.weather-page .stat-card .v-icon {
  transition: color 0.25s ease;
}

.weather-page .stat-card:hover .v-icon {
  color: #b71c1c;
}

/* Individual icon animations */
.stat-col:nth-child(1) .stat-card:hover .v-icon {
  animation: drop-bounce 0.9s ease-in-out infinite;
}

.stat-col:nth-child(2) .stat-card:hover .v-icon {
  animation: wind-sway 1s ease-in-out infinite;
}

.stat-col:nth-child(3) .stat-card:hover .v-icon {
  animation: rain-wobble 0.7s ease-in-out infinite;
}

.stat-col:nth-child(4) .stat-card:hover .v-icon {
  animation: sun-rise 1.2s ease-in-out infinite;
}

/* Animations */
@keyframes drop-bounce {

  0%,
  100% {
    transform: translateY(0) scale(1);
  }

  50% {
    transform: translateY(-6px) scale(1.15);
  }
}

@keyframes wind-sway {

  0%,
  100% {
    transform: translateX(-4px);
  }

  50% {
    transform: translateX(4px);
  }
}

@keyframes rain-wobble {

  0%,
  100% {
    transform: rotate(-8deg);
  }

  50% {
    transform: rotate(8deg);
  }
}

@keyframes sun-rise {

  0%,
  100% {
    transform: translateY(3px) scale(1);
  }

  50% {
    transform: translateY(-5px) scale(1.2);
  }
}

/* Mobile layout */
@media (max-width: 600px) {
  .hero-temp {
    font-size: 56px;
  }

  .day-row {
    grid-template-columns: 1fr 30px 45px 75px;
    gap: 6px;
  }
}

/* Respect reduced-motion settings */
@media (prefers-reduced-motion: reduce) {

  .weather-page .v-card.stat-card:hover,
  .stat-col .stat-card:hover .v-icon {
    animation: none;
    transform: none;
  }
}
</style>