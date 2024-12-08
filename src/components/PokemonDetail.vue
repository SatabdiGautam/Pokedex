<template>
  <!-- Navbar -->
  <nav
    class="bg-red-600 text-white py-4 px-6 fixed w-full top-0 z-10 shadow-md border-b-2 border-black"
  >
    <div class="flex justify-center items-center w-full">
      <h1 class="text-2xl font-bold">{{ pokemon.name }}</h1>
    </div>
  </nav>

  <!-- Body with Pokemon details -->
  <body class="pt-24 relative">
    <div class="p-4 relative">
      <!-- Pokemon Details -->
      <div
        v-if="!isLoading"
        class="text-center max-w-4xl mx-auto bg-gray-100 p-6 rounded-lg shadow-lg"
      >
        <div class="relative">
          <!-- Pokémon ID -->
          <div
            class="absolute top-4 right-4 bg-gray-200 text-gray-700 px-4 py-2 rounded-lg shadow-lg"
          >
            <p class="text-lg font-semibold">#{{ pokemon.id }}</p>
          </div>

          <!-- Pokémon Image -->
          <div class="flex justify-center mb-6">
            <img :src="pokemon.image" alt="Pokemon" class="w-48 h-48" />
          </div>
        </div>

        <!-- General Info -->
        <div class="grid grid-cols-2 gap-4 mb-6">
          <div class="bg-white p-4 rounded-lg shadow-md">
            <h2 class="font-bold text-lg">Height</h2>
            <p class="text-gray-700">{{ formatHeight(pokemon.height) }}</p>
          </div>
          <div class="bg-white p-4 rounded-lg shadow-md">
            <h2 class="font-bold text-lg">Weight</h2>
            <p class="text-gray-700">{{ pokemon.weight.toFixed(1) }} kg</p>
          </div>
          <div class="bg-white p-4 rounded-lg shadow-md">
            <h2 class="font-bold text-lg">Types</h2>
            <p class="text-gray-700">{{ pokemon.types.join(", ") }}</p>
          </div>
          <div class="bg-white p-4 rounded-lg shadow-md">
            <h2 class="font-bold text-lg">Abilities</h2>
            <p class="text-gray-700">{{ pokemon.abilities.join(", ") }}</p>
          </div>
        </div>

        <!-- Base Stats -->
        <div class="bg-white p-6 rounded-lg shadow-md mb-6">
          <h2 class="font-bold text-xl mb-4">Base Stats</h2>
          <div class="space-y-4">
            <div
              v-for="(stat, index) in pokemon.stats"
              :key="index"
              class="flex items-center space-x-4"
            >
              <!-- Stat Name -->
              <p class="capitalize w-40 text-gray-700 font-semibold truncate">
                {{ stat.stat.name }}
              </p>
              <!-- Stat Bar -->
              <div class="flex-1 bg-gray-200 h-6 rounded-lg overflow-hidden">
                <div
                  class="bg-red-600 h-full text-center text-white text-sm font-semibold"
                  :style="{ width: (stat.base_stat / maxStat) * 100 + '%' }"
                >
                  {{ stat.base_stat }}
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- Moves -->
        <div class="bg-white p-6 rounded-lg shadow-md">
          <h2 class="font-bold text-xl mb-4">Moves</h2>
          <div class="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 gap-2">
            <div
              v-for="(move, index) in pokemon.moves"
              :key="index"
              class="bg-gray-100 p-2 rounded-md text-center text-gray-700 shadow-sm"
            >
              {{ move }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </body>
</template>

<script setup>
import { ref, onMounted } from "vue";
import { useRoute } from "vue-router";

const route = useRoute();
const pokemon = ref({
  id: null,
  name: "",
  image: "",
  height: null,
  weight: null,
  types: [],
  abilities: [],
  moves: [],
  stats: [],
});
const isLoading = ref(true);
const maxStat = 200; // Maximum stat value for the progress bar scaling

// Convert decimeters to feet and inches
const formatHeight = (heightInDecimeters) => {
  const totalInches = heightInDecimeters * 3.937; // 1 decimeter = 3.937 inches
  const feet = Math.floor(totalInches / 12);
  const inches = Math.round(totalInches % 12);
  return `${feet}'${inches}"`;
};

// Fetch Pokémon details on mount
onMounted(async () => {
  const response = await fetch(
    `https://pokeapi.co/api/v2/pokemon/${route.params.id}`
  );
  const data = await response.json();
  pokemon.value = {
    id: data.id,
    name: data.name,
    image: data.sprites.front_default,
    height: data.height, // in decimeters
    weight: data.weight / 10, // Convert to kilograms
    types: data.types.map((type) => type.type.name), // Map types to an array
    abilities: data.abilities.map((ability) => ability.ability.name), // Map abilities to an array
    moves: data.moves.map((move) => move.move.name), // Map moves to an array
    stats: data.stats, // Stats are already in the required format
  };
  isLoading.value = false; // Stop loading once data is fetched
});
</script>