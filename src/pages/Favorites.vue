<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

import CardList from '../components/CardList.vue'

const favorites = ref([]);

onMounted(async () => {
    try {
        const { data } = await axios.get('https://d5e1d70c5e113cd5.mokky.dev/favorites');
        favorites.value = data.map(obj => obj.item);
        console.log(favorites)
    } catch (err) {
        console.log(err)
    }
})
</script>

<template>
  <h2 class="text-3xl font-bold mb-8">Мои закладки</h2>

  <CardList :items="favorites"/>
</template>
