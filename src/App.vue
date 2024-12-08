<template>
  <div class="container">
    <LoadingScreen v-if="isLoading" />
    <router-view v-else :pokemon-list="pokemonList" />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import LoadingScreen from './components/LoadingScreen.vue';

const isLoading = ref(true);
const pokemonList = ref([]);

const fetchData = async () => {
  try {
    let allPokemons = [];
    let nextUrl = 'https://pokeapi.co/api/v2/pokemon?limit=100';

    while (nextUrl) {
      const response = await fetch(nextUrl);
      const data = await response.json();

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
      nextUrl = data.next;
    }

    pokemonList.value = allPokemons;
  } catch (error) {
    console.error('Error fetching data:', error);
  } finally {
    isLoading.value = false;
  }
};

onMounted(() => {
  fetchData();
});
</script>