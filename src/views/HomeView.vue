<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or town"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow=[0px_1px_0_0_#004E71]"
      />
      <ul
        v-if="searchQuery"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      >
        <p v-if="searchError">
          Sorry, something went wrong. Please try again later.
        </p>
        <p v-if="!searchError && openweathermapSearchResults.length === 0">
          No results found.
        </p>
        <template v-else>
          <li
            v-for="searchResult in openweathermapSearchResults"
            :key="searchResult.id"
            class="py-2 cursor-pointer"
            @click="previewCity(searchResult)"
          >
            {{
              searchResult.name +
              ", " +
              (searchResult.state ? searchResult.state + ", " : "") +
              regionNames.of(searchResult.country)
            }}
          </li>
        </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList/>
        <template #fallback>
          <p>Loading...</p> 
        </template>
      </Suspense>
    </div>
  </main>
</template>
<script setup>
import { ref } from "vue";
import axios from "axios";
import { useRouter } from "vue-router";
import CityList from "../components/CityList.vue";

const router = useRouter();
const previewCity = (searchResult) => {
  console.log(searchResult);
  router.push({
    name: 'cityView',
    params: {state: searchResult.state, city: searchResult.name},
    query: {
      lat: searchResult.lat,
      lon: searchResult.lon,
      preview: true
    }
  })
};

const openweathermapAPIKey = "9ed557cdf7c9ae9ddaf9a3ec13532116";
const searchQuery = ref("");
const queryTimeout = ref(null);
const openweathermapSearchResults = ref(null);
const searchError = ref(null);

const regionNames = new Intl.DisplayNames(["en"], { type: "region" });

const getSearchResults = () => {
  queryTimeout.value = setTimeout(async () => {
    clearTimeout(queryTimeout.value);
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `http://api.openweathermap.org/geo/1.0/direct?q=${searchQuery.value}&limit=5&appid=${openweathermapAPIKey}`
        );
        openweathermapSearchResults.value = result.data;
        console.log(openweathermapSearchResults.value);
        return;
      } catch {
        searchError.value = true;
      }
    }
    openweathermapSearchResults = null;
  }, 300);
};
</script>
