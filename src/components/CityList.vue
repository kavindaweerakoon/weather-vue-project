<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>
  <p v-if="savedCities.length === 0">
    No locations added. To start tracking a location, search in the field above.
  </p>
</template>

<script setup>
import axios from "axios";
import { ref } from "vue";
import { useRouter } from "vue-router";
import CityCard from "./CityCard.vue";

const savedCities = ref([]);
const getCities = async () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));
  }

  const requests = [];
  const openweathermapAPIKey = "9ed557cdf7c9ae9ddaf9a3ec13532116";
  savedCities.value.forEach((city) => {
    requests.push(
      axios.get(
        `https://api.openweathermap.org/data/2.5/weather?lat=${city.coords.lat}&lon=${city.coords.lon}&appid=${openweathermapAPIKey}&units=metric`
      )
    );
  });
  const weatherData = await Promise.all(requests);

  await new Promise ((res) => setTimeout(res, 500))

  weatherData.forEach((value, index) => {
    savedCities.value[index].weather = value.data;
  });
};
await getCities();

const router = useRouter();
const goToCityView = (city) => {
  router.push({
    name: 'cityView',
    params: {state: city.state, city: city.city},
    query: {id: city.id, lat: city.coords.lat, lon: city.coords.lon}
  })
};
</script>
