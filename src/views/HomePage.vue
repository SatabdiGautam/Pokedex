<template>
  <div>
    <nav class="bg-red-600 text-white py-4 px-6 fixed w-full top-0 z-10 shadow-md border-b-2 border-black">
      <div class="flex flex-wrap justify-between items-center">
        <h1 class="text-2xl font-bold w-full sm:w-auto">Pokedex</h1>
        <div class="relative w-full sm:w-1/2 md:w-1/3 mt-2 sm:mt-0">
          <input
            v-model="filterName"
            type="text"
            placeholder="Search"
            class="w-full pl-10 pr-4 py-2 rounded-full border-2 border-red-500 focus:outline-none focus:ring-2 focus:ring-red-400 text-black placeholder-gray-500"
          />
          <img
            src="../assets/searchIcon.svg"
            alt="Search"
            class="absolute left-3 top-1/2 transform -translate-y-1/2 w-5 h-5 text-red-500"
          />
        </div>
      </div>
    </nav>
    <div class="container mx-auto pt-24 grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6 max-w-full">
      <div
        v-for="pokemon in filteredPokemon"
        :key="pokemon.id"
        class="bg-white rounded-lg shadow-md hover:shadow-lg transition-shadow border p-4 text-center relative"
      >
        <div class="absolute top-1/2 left-0 right-0 h-1 bg-red-600"></div>

        <router-link :to="'/pokemon/' + pokemon.id" class="block relative">
          <img
            :src="pokemon.image"
            alt="Pokemon"
            class="w-32 h-32 mx-auto mb-2"
          />
          <p class="font-semibold text-lg capitalize text-gray-700">
            {{ pokemon.name }}
          </p>
        </router-link>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import '../../index.css';

const filterName = ref('');
const pokemonList = ref([]);
const isLoading = ref(true);

// Filter Pokémon based on search input
const filteredPokemon = computed(() => {
  return pokemonList.value.filter((pokemon) =>
    pokemon.name.toLowerCase().includes(filterName.value.toLowerCase())
  );
});

// Function to fetch all Pokémon data
const fetchAllPokemon = async () => {
  let allPokemons = [];
  let nextUrl = 'https://pokeapi.co/api/v2/pokemon?limit=100'; // Start with the first page

  while (nextUrl) {
    try {
      const response = await fetch(nextUrl);
      const data = await response.json();

      // For each Pokémon, fetch more details
      const pokemonDetails = await Promise.all(
        data.results.map(async (pokemon) => {
          const detailResponse = await fetch(pokemon.url);
          const detailData = await detailResponse.json();
          return {
            id: detailData.id,
            name: detailData.name,
            image: detailData.sprites.front_default,
          };
        })
      );

      allPokemons = [...allPokemons, ...pokemonDetails];
      nextUrl = data.next; // Get the next page if available
    } catch (error) {
      console.error('Error fetching Pokémon data:', error);
      nextUrl = null; // Stop if there's an error
    }
  }

  pokemonList.value = allPokemons;
  isLoading.value = false; // Data loading complete
};

// Fetch all Pokémon when the component is mounted
onMounted(() => {
  fetchAllPokemon();
});
</script>