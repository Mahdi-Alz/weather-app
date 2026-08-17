<script setup>
import { reactive } from "vue";

const emit = defineEmits(["place-data"]);

const API_KEY = import.meta.env.VITE_WEATHER_API_KEY;

const searchTerm = reactive({
  query: "",
  timeout: null,
  results: null,
  loading: false,
  error: null,
});

const handleSearch = () => {
  clearTimeout(searchTerm.timeout);
  searchTerm.error = null;

  if (!searchTerm.query.length) {
    searchTerm.results = null;
    searchTerm.loading = false;
    return;
  }

  searchTerm.loading = true;
  searchTerm.timeout = setTimeout(async () => {
    try {
      const res = await fetch(
        `https://api.weatherapi.com/v1/search.json?key=${API_KEY}&q=${encodeURIComponent(searchTerm.query)}`,
      );
      if (!res.ok) throw new Error("search failed");
      searchTerm.results = await res.json();
    } catch {
      searchTerm.results = [];
      searchTerm.error = "Couldn't reach the weather service. Try again.";
    } finally {
      searchTerm.loading = false;
    }
  }, 500);
};

const clearSearch = () => {
  clearTimeout(searchTerm.timeout);
  searchTerm.query = "";
  searchTerm.results = null;
  searchTerm.error = null;
  searchTerm.loading = false;
};

const getWeather = async (id) => {
  const res = await fetch(
    `https://api.weatherapi.com/v1/forecast.json?key=${API_KEY}&q=id:${id}&days=3&aqi=no&alerts=no`,
  );
  const data = await res.json();
  emit("place-data", data);
  clearSearch();
};
</script>

<template>
  <div class="relative mx-auto w-full max-w-xl">
    <form @submit.prevent>
      <label
        class="input input-lg w-full items-center gap-3 rounded-box border-base-300 bg-base-200/80 shadow-lg shadow-black/20 backdrop-blur focus-within:border-primary/60"
      >
        <svg
          class="size-5 shrink-0 text-base-content/50"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
        >
          <circle cx="11" cy="11" r="7" />
          <path d="m20 20-3.5-3.5" />
        </svg>

        <input
          type="text"
          placeholder="Search for a city…"
          class="grow"
          v-model="searchTerm.query"
          @input="handleSearch"
        />

        <span v-if="searchTerm.loading" class="loading loading-spinner loading-sm text-base-content/40"></span>
        <button
          v-else-if="searchTerm.query"
          type="button"
          class="btn btn-ghost btn-circle btn-xs text-base-content/50 hover:text-base-content"
          @click="clearSearch"
          aria-label="Clear search"
        >
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
            <path d="M18 6 6 18M6 6l12 12" />
          </svg>
        </button>
      </label>
    </form>

    <!-- search suggestions -->
    <ul
      v-if="searchTerm.results !== null"
      class="menu absolute z-10 mt-2 w-full rounded-box border border-base-300 bg-base-200/95 p-2 shadow-xl shadow-black/30 backdrop-blur"
    >
      <li v-if="searchTerm.error" class="px-3 py-2 text-sm text-error">
        {{ searchTerm.error }}
      </li>
      <li v-else-if="searchTerm.results.length === 0" class="px-3 py-2 text-sm text-base-content/50">
        No places found.
      </li>
      <li v-for="place in searchTerm.results" :key="place.id">
        <button type="button" class="rounded-field" @click="getWeather(place.id)">
          <svg
            class="size-4 shrink-0 text-base-content/40"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
            stroke-linejoin="round"
          >
            <path d="M12 21s-7-6.1-7-11a7 7 0 0 1 14 0c0 4.9-7 11-7 11Z" />
            <circle cx="12" cy="10" r="2.5" />
          </svg>
          <span class="truncate">{{ place.name }}<span class="text-base-content/50">, {{ place.region }}, {{ place.country }}</span></span>
        </button>
      </li>
    </ul>
  </div>
</template>
