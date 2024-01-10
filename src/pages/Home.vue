<script setup>
import { reactive, watch, ref, onMounted } from 'vue'
import debounce from 'lodash.debounce'
import axios from 'axios'
import { inject } from 'vue';
import CardList from '../components/CardList.vue'

const { cart, addToCart, removeFromCart } = inject('cart');

const items = ref([]);

const filters = reactive({
  sortBy: 'title',
  searchQuery: ''
});

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
      parentId: item.id,
      item
    }

    item.isFavorite = true;
    const { data } = await axios.post('https://d5e1d70c5e113cd5.mokky.dev/favorites', obj);

    item.favoriteId = data.id;
    } else {
      item.isFavorite = false;
      await axios.delete(`https://d5e1d70c5e113cd5.mokky.dev/favorites/${item.favoriteId}`)
      item.favoriteId = null;
    }


  } catch (err) {
    console.log(err)
  }
}

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }

  console.log(cart)
}

//Функция, кторорая следит за изменениями в поле поиска (нужно посмотреть про debounce)
const onChangeSearcInput = debounce((event) => { 
  filters.searchQuery = event.target.value
}, 500)

const onChangeSelect = (event) => {   //Функция, которая следит за изменениями в списке
  filters.sortBy = event.target.value
}

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get('https://d5e1d70c5e113cd5.mokky.dev/favorites');
    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.parentId === item.id) //Поиск одинаковых элементов в массиве всех товаров и в массиве закладок

      if(!favorite) {    //Если таких нет, возвращаем сам объект
        return item;
      }

      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id,
      }
    });

  } catch (err) {
    console.log(err)
  }
}

const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    };

    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`;
    }
    const { data } = await axios.get('https://d5e1d70c5e113cd5.mokky.dev/items', {
      params
    });
    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false
    }));
  } catch (err) {
    console.log(err)
  }
}

//Отпрвка запроса на бэк 
onMounted(async () => {
  const localCart = localStorage.getItem('cart');
  cart.value = localCart ? JSON.parse(localCart) : [];

  await fetchItems();
  await fetchFavorites();

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }))
})

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false
  }))
})

watch(filters, fetchItems);
</script>

<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>

    <div class="flex gap-4">
      <select
        @change="onChangeSelect"
        class="py-2 px-3 border rounded-md outline-none"
        name=""
        id=""
      >
        <option value="name">По названию</option>
        <option value="price">По цене (дешёвые)</option>
        <option value="-price">По цене (дорогие)</option>
      </select>

      <div class="relative">
        <img class="absolute left-3 top-3" src="/search.svg" alt="" />
        <input
          @input="onChangeSearcInput"
          class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
          type="text"
          placeholder="Поиск..."
        />
      </div>
    </div>
  </div>

  <CardList :items="items" @add-to-favorite="addToFavorite" @add-to-cart="onClickAddPlus" />
</template>
