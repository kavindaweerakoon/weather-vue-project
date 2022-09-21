<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input
        type="text"
        v-model="searchQuery"
        @input="getSearchResults"
        placeholder="Search for a city or state/province"
        class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none focus:shadow=[0px_1px_0_0_#004E71]"
      />
      <ul
      v-if="searchQuery"
        class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]"
      >
        <li
          
          v-for="searchResult in openweathermapSearchResults"
          :key="searchResult.id"
          class="py-2 cursor-pointer"
        >
          {{
            searchResult.name +
            ", " +
            (searchResult.state ? searchResult.state + ", " : "") +
            regionNames.of(searchResult.country)
          }}
        </li>
      </ul>
    </div>
  </main>
</template>
<script setup>
import { ref } from "vue";
import axios from "axios";

const openweathermapAPIKey = "9ed557cdf7c9ae9ddaf9a3ec13532116";
const searchQuery = ref("");
const queryTimeout = ref(null);
let openweathermapSearchResults = ref(null);

const regionNames = new Intl.DisplayNames(["en"], { type: "region" });

const getSearchResults = () => {
  queryTimeout.value = setTimeout(async () => {
    clearTimeout(queryTimeout.value);
    if (searchQuery.value !== "") {
      const result = await axios.get(
        `http://api.openweathermap.org/geo/1.0/direct?q=${searchQuery.value}&limit=5&appid=${openweathermapAPIKey}`
      );
      openweathermapSearchResults.value = result.data;
      console.log(openweathermapSearchResults.value);
      return;
    }
    openweathermapSearchResults = null;
  }, 300);
};
</script>
