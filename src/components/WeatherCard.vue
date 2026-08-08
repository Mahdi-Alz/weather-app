<script setup>
import { computed } from "vue";

const props = defineProps({
  place: { type: Object, required: true },
});
const emit = defineEmits(["remove"]);

const icon = (path) => `https:${path}`;

const isDay = computed(() => props.place.current.is_day === 1);

const skyGradient = computed(() =>
  isDay.value
    ? "linear-gradient(160deg, #16233f 0%, #2c4a86 55%, #6c8cff 100%)"
    : "linear-gradient(160deg, #05070d 0%, #0b1220 55%, #1b2740 100%)",
);

const upcoming = computed(() => props.place.forecast?.forecastday ?? []);

const weekday = (date) =>
  new Date(date).toLocaleDateString("en-us", { weekday: "short" });
</script>

<template>
  <article class="card overflow-hidden rounded-box border border-base-300 bg-base-200 shadow-lg shadow-black/20">
    <!-- sky band -->
    <div class="relative flex items-center gap-4 p-5" :style="{ background: skyGradient }">
      <div
        v-if="isDay"
        class="pointer-events-none absolute -right-6 -top-6 size-28 rounded-full bg-accent/30 blur-2xl"
      ></div>
      <button
        type="button"
        class="btn btn-ghost btn-circle btn-xs absolute right-3 top-3 text-white/70 hover:bg-white/10 hover:text-white"
        @click="emit('remove')"
        aria-label="Remove place"
      >
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
          <path d="M18 6 6 18M6 6l12 12" />
        </svg>
      </button>

      <img :src="icon(place.current.condition.icon)" :alt="place.current.condition.text" class="size-16 shrink-0 drop-shadow" />
      <div class="min-w-0">
        <h2 class="truncate font-display text-lg font-semibold text-white">{{ place.location.name }}</h2>
        <p class="truncate text-xs text-white/60">{{ place.location.region ? place.location.region + ", " : "" }}{{ place.location.country }}</p>
        <p class="mt-1 text-sm text-white/80">{{ place.current.condition.text }}</p>
      </div>
    </div>

    <div class="card-body gap-4 p-5">
      <!-- temperature -->
      <div class="flex items-end justify-between">
        <div class="font-display text-6xl font-bold leading-none tracking-tight">
          {{ Math.round(place.current.temp_c) }}<span class="text-3xl text-base-content/50">°C</span>
        </div>
        <div class="text-right text-sm text-base-content/60">
          Feels like<br />
          <span class="font-mono text-base text-base-content/80">{{ Math.round(place.current.feelslike_c) }}°</span>
        </div>
      </div>

      <!-- stats -->
      <div class="grid grid-cols-3 gap-3 rounded-field border border-base-300 bg-base-100/60 p-3 font-mono text-xs">
        <div>
          <div class="text-base-content/40">HUMIDITY</div>
          <div class="mt-0.5 text-sm text-base-content/90">{{ place.current.humidity }}%</div>
        </div>
        <div>
          <div class="text-base-content/40">WIND</div>
          <div class="mt-0.5 text-sm text-base-content/90">{{ Math.round(place.current.wind_kph) }} km/h</div>
        </div>
        <div>
          <div class="text-base-content/40">UV</div>
          <div class="mt-0.5 text-sm text-base-content/90">{{ place.current.uv }}</div>
        </div>
      </div>

      <!-- 3-day forecast -->
      <div class="flex items-center justify-between border-t border-base-300 pt-3">
        <div
          v-for="day in upcoming"
          :key="day.date"
          class="flex flex-1 flex-col items-center gap-1"
        >
          <span class="text-xs text-base-content/50">{{ weekday(day.date) }}</span>
          <img :src="icon(day.day.condition.icon)" :alt="day.day.condition.text" class="size-8" />
          <span class="font-mono text-xs">
            <span class="text-base-content/90">{{ Math.round(day.day.maxtemp_c) }}°</span>
            <span class="text-base-content/40">/{{ Math.round(day.day.mintemp_c) }}°</span>
          </span>
        </div>
      </div>
    </div>
  </article>
</template>
