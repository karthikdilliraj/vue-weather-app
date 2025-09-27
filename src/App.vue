<script setup lang="ts">
import { ref, onMounted } from 'vue'
import Card from '@/components/ui/card/Card.vue'
import CardContent from '@/components/ui/card/CardContent.vue'
import Skeleton from '@/components/ui/skeleton/Skeleton.vue'
import Input from '@/components/ui/input/Input.vue'
import Button from '@/components/ui/button/Button.vue'
// Icons (you can use lucide-vue-next or similar)
import { MapPinIcon, } from 'lucide-vue-next'

interface Data {
  place: string;
  temp: string;
  description: string;
  icon: string;
  humidity: string;
  wind: string;
  feels_like: string;
  rain_chance: string;
  forecast: {
    avgTemp: string;
    condition: string;
    date: string;
    day: string;
    icon: string;
  }[];
}

interface Suggestion {
  name: string;
  lat: string;
  lon: string;
  country: string;
  state?: string;
}

const weather = ref<Data>({
  place: "Downtown Toronto, CA",
  temp: "26°C",
  description: "Sunny",
  icon: "https://openweathermap.org/img/wn/01d@2x.png",
  humidity: "60%",
  wind: "12 km/h",
  feels_like: "28°C",
  rain_chance: "0%",
  forecast: [],
})
const search = ref("")
const suggestions = ref<Suggestion[]>([])
const loading = ref(true)

// simulate loading
onMounted(() => {
  navigator.geolocation.getCurrentPosition(
    (geolocationPosition) => {
      const { coords } = geolocationPosition;
      fetch(
        `${buildUrl()}api/data?lat=${coords.latitude}&lon=${coords.longitude}`
      )
        .then((res) => {
          return res.json();
        })
        .then((data) => {
          weather.value = data
          loading.value = false
        });
    },
    () => {
      initialFetcher();
    }
  );
})

const initialFetcher = () => {
  const searchQuery = search.value ? search.value : "Toronto";
  loading.value = true
  fetch(`${buildUrl()}api/data?city=${searchQuery}`)
    .then((res) => {
      return res.json();
    })
    .then((data) => {
      const suggestion = data[0];
      fetcher(suggestion);
    });
};

const fetcher = (suggestion: Suggestion) => {
  fetch(`${buildUrl()}api/data?lat=${suggestion.lat}&lon=${suggestion.lon}`)
    .then((res) => {
      return res.json();
    })
    .then((data) => {
      weather.value = data
      loading.value = false
      if (search.value) {
        search.value = ""
      }
    });
};

function buildUrl() {
  const url = new URL(window.location.protocol + "//" + window.location.host);
  return url.href;
}


const updateSuggestions = () => {
  if (search.value.length > 0) {
    fetch(`${buildUrl()}api/data?city=${search.value}`)
      .then((res) => {
        return res.json();
      })
      .then((data) => {
        suggestions.value = data
      });
  } else {
    suggestions.value = []

  }
};
const handleSelectCity = (suggestion: Suggestion) => {
  fetcher(suggestion);
  suggestions.value = []
};

function getSuggestionLabel(suggestion: Suggestion) {
  return [suggestion.name, suggestion.state, suggestion.country]
    .filter(Boolean)
    .join(", ")
}
</script>

<template>
  <div class="min-h-screen bg-gradient-to-b from-blue-500 to-indigo-700 flex flex-col items-center p-6">
    <!-- Header -->
    <h1 class="text-white text-3xl font-bold mb-6">WeatherNow</h1>

    <!-- Search with Input (autocomplete) -->
    <div class="relative w-full max-w-md mb-6">
      <div class="flex gap-2">
        <Input placeholder="Enter city name..." class="rounded-2xl bg-white" v-model="search" />
        <Button class="rounded-2xl" @click="updateSuggestions">
          Search
        </Button>
      </div>
      <template v-if="suggestions.length > 0">
        <div class="absolute top-12 left-0 w-full bg-white rounded-xl shadow-lg z-10">
          <template v-for="suggestion, index of suggestions" :key="index">
            <div class="p-2 cursor-pointer hover:bg-gray-100 rounded-xl" @click="handleSelectCity(suggestion)" v-text="getSuggestionLabel(suggestion)
              ">
            </div>
          </template>

        </div>
      </template>
    </div>


    <div class="w-full max-w-md">
      <template v-if="loading">
        <Card class="rounded-2xl shadow-lg bg-white/70 backdrop-blur">
          <CardContent class="p-6 flex flex-col items-center text-center space-y-4">
            <Skeleton class="h-6 w-24" />
            <Skeleton class="h-16 w-16 rounded-full" />
            <Skeleton class="h-12 w-32" />
            <Skeleton class="h-4 w-20" />
            <div class="grid grid-cols-2 gap-4 w-full mt-6">
              <template v-for="_, i of [...Array(4)]" :key="i">
                <Skeleton class="h-16 w-full rounded-xl" />
              </template>

            </div>
          </CardContent>
        </Card>
      </template>
      <template v-else>
        <Card class="rounded-2xl shadow-lg bg-white/70 backdrop-blur">
          <CardContent class="p-6 flex flex-col items-center text-center">
            <MapPinIcon class="text-blue-600" />
            <h2 class="text-xl font-semibold" v-text="weather.place"></h2>
            <div class="my-4">
              <img :src="weather.icon" alt="Weather Icon" class="w-16 h-16" />
            </div>
            <p class="text-5xl font-bold" v-text="weather.temp"></p>
            <p class="text-gray-600 capitalize" v-text="weather.description"></p>

            <div class="grid grid-cols-2 gap-4 mt-6 w-full">
              <div class="bg-blue-100 p-3 rounded-xl">
                <p class="text-gray-600 text-sm">Humidity</p>
                <p class="text-lg font-bold" v-text="weather.humidity"></p>
              </div>
              <div class="bg-blue-100 p-3 rounded-xl">
                <p class="text-gray-600 text-sm">Wind</p>
                <p class="text-lg font-bold" v-text="weather.wind"></p>
              </div>
              <div class="bg-blue-100 p-3 rounded-xl">
                <p class="text-gray-600 text-sm">Feels Like</p>
                <p class="text-lg font-bold" v-text="weather.feels_like"></p>
              </div>
              <div class="bg-blue-100 p-3 rounded-xl">
                <p class="text-gray-600 text-sm">Rain Chance</p>
                <p class="text-lg font-bold" v-text="weather.rain_chance"></p>
              </div>
            </div>
          </CardContent>
        </Card>
      </template>
    </div>

    <!-- Forecast -->
    <div class="mt-8 w-full max-w-md">
      <h3 class="text-white text-lg font-semibold mb-4">
        5-Day Forecast
      </h3>
      <div class="grid grid-cols-5 gap-3">
        <template v-for="f, i of weather.forecast" :key="i">
          <Card class="rounded-xl bg-white/80 backdrop-blur text-center">
            <CardContent class="p-3 flex flex-col items-center">
              <p class="font-semibold" v-text="f.day"></p>
              <img :src="f.icon" alt="Weather Icon" class="w-8 h-8" />
              <p class="font-bold" v-text="f.avgTemp"></p>
            </CardContent>
          </Card>
        </template>
      </div>
    </div>
  </div>
</template>
