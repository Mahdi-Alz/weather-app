<script setup>
import { ref } from "vue";
import SearchInput from "./components/SearchInput.vue";

const places = ref([]);
const addPlace = (data) => {
  places.value.push(data);
};
</script>

<template>
  <main class="min-h-dvh bg-zinc-100 p-8">
    <h1 class="text-center">Weather Application</h1>
    <div class="text-center mb-6">
      {{
        new Date().toLocaleDateString("en-us", {
          weekday: "long",
          year: "numeric",
          month: "long",
          day: "numeric",
        })
      }}
    </div>
    <!-- Search -->
    <div>
      <SearchInput @place-data="addPlace" />
    </div>
    <!-- weather cards -->
    <div>
      <p v-for="place in places">{{ place.location.name }}</p>
    </div>
  </main>
</template>
