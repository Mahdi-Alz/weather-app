<script setup>
import { reactive } from "vue";

const emit = defineEmits(["place-data"]);

const searchTerm = reactive({
  query: "",
  timeout: null,
  results: null,
});
const handleSearch = () => {
  clearTimeout(searchTerm.timeout);
  searchTerm.timeout = setTimeout(async () => {
    if (searchTerm.query.length) {
      const res = await fetch(
        `http://api.weatherapi.com/v1/search.json?key=5805c27e61e94d369c674729262306&q=${searchTerm.query}`,
      );
      const data = await res.json();
      searchTerm.results = data;
    } else {
      searchTerm.results = null;
    }
  }, 500);
};

const getWeather = async (id) => {
  const res = await fetch(
    `http://api.weatherapi.com/v1/forecast.json?key=5805c27e61e94d369c674729262306&q=id:${id}&days=3&aqi=no&alerts=no`,
  );
  const data = await res.json();
  emit("place-data", data);
  searchTerm.results = null;
  searchTerm.query = "";
};
</script>

<template>
  <div>
    <!-- search field -->
    <form>
      <div
        class="bg-white border border-indigo-600/30 rounded-lg shadow-lg flex items-center"
      >
        <span
          class="flex items-center justify-center p-2 shrink-0 text-indigo-600"
        >
          <i class="fa-solid fa-magnifying-glass"></i>
        </span>

        <input
          type="text"
          placeholder="Search for a place"
          class="rounded-r-lg p-2 border-0 outline-0 focus:ring-2 focus:ring-indigo-600 ring-inset w-full"
          v-model="searchTerm.query"
          @input="handleSearch"
        />
      </div>
    </form>
    <!-- search suggestions -->
    <div class="bg-white my-2 rounded-lg shadow-lg">
      <div v-if="searchTerm.results">
        <div v-for="place in searchTerm.results" :key="place.id">
          <button
            class="px-3 my-2 hover:text-indigo-600 hover:font-bold transition-all duration-200 ease-in-out w-full text-left"
            @click="getWeather(place.id)"
          >
            {{ place.name }}, {{ place.region }}, {{ place.country }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>
