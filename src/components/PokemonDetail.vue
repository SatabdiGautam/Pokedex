<template>
  <!-- Navbar -->
  <nav class="bg-red-600 text-white py-4 px-6 fixed w-full top-0 z-10 shadow-md border-b-2 border-black">
    <div class="flex justify-center items-center w-full">
      <h1 class="text-2xl font-bold">{{ pokemon.name }}</h1>
    </div>
  </nav>

  <!-- Body with Pokemon details -->
  <body class="pt-24">
    <div class="p-4">
      <!-- Loading Spinner -->
      <div v-if="isLoading" class="flex justify-center items-center">
        <svg xmlns="http://www.w3.org/2000/svg" class="animate-spin h-16 w-16 text-red-600" viewBox="0 0 24 24" fill="none" stroke="currentColor">
          <circle cx="12" cy="12" r="10" stroke-width="4" class="opacity-25" />
          <path d="M4 12a8 8 0 1 1 8 8" stroke-linecap="round" stroke-linejoin="round" stroke-width="4" class="opacity-75"/>
        </svg>
      </div>

      <!-- Pokemon Details -->
      <div v-if="!isLoading" class="text-center">
        <div class="flex justify-center mb-4">
          <img :src="pokemon.image" alt="Pokemon" class="w-48 h-48"/>
        </div>
        <div class="space-y-2">
          <p><strong>ID:</strong> {{ pokemon.id }}</p>
          <p><strong>Height:</strong> {{ pokemon.height }} decimeters</p>
          <p><strong>Weight:</strong> {{ pokemon.weight }} hectograms</p>
          <p><strong>Types:</strong> {{ pokemon.types.join(', ') }}</p>
          <p><strong>Abilities:</strong> {{ pokemon.abilities.join(', ') }}</p>
          <p><strong>Moves:</strong> {{ pokemon.moves.slice(0, 5).join(', ') }}...</p>
          <p><strong>Base Stats:</strong></p>
          <ul class="list-disc text-left mx-6">
            <li v-for="(stat, index) in pokemon.stats" :key="index">
              {{ stat.stat.name }}: {{ stat.base_stat }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </body>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();
const pokemon = ref({
  id: null,
  name: '',
  image: '',
  height: null,
  weight: null,
  types: [],
  abilities: [],
  moves: [],
  stats: []
});
const isLoading = ref(true);  // Keeps track of whether the image and data are loading

// Fetch Pokémon details on mount
onMounted(async () => {
  const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${route.params.id}`);
  const data = await response.json();
  pokemon.value = {
    id: data.id,
    name: data.name,
    image: data.sprites.front_default,
    height: data.height,
    weight: data.weight,
    types: data.types.map(type => type.type.name),  // Map types to an array
    abilities: data.abilities.map(ability => ability.ability.name),  // Map abilities to an array
    moves: data.moves.map(move => move.move.name),  // Map moves to an array
    stats: data.stats,  // Stats are already in the required format
  };
  isLoading.value = false;  // Stop loading once data is fetched
});
</script>