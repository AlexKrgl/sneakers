<script setup>
defineProps({
  totalPrice: Number
})
const emit = defineEmits(['openDrawer'])
</script>

<template>
  <header class="flex justify-between border-b border-slate-300 px-10 py-8">
    <router-link to="/"
      ><div class="flex items-center gap-4">
        <img src="/logo.png" class="w-10" alt="Logo" />
        <div>
          <h2 class="text-xl font-bold uppercase">Vue Sneakers</h2>
          <p class="text-slate-400">Магазин лучших кроссовок</p>
        </div>
      </div></router-link
    >

    <ul class="flex items-center gap-10">
      <li
        @click="emit('openDrawer')"
        class="flex items-center gap-3 text-gray-500 hover:text-black cursor-pointer"
      >
        <img src="/cart.svg" alt="cart" />
        <b>{{ totalPrice }} руб.</b>
      </li>


      <router-link to="/favorites">
        <li class="flex items-center gap-3 text-gray-500 hover:text-black cursor-pointer">
          <img src="/heart.svg" alt="hert" />
          <span>Закладки</span>
        </li>
      </router-link>

      <li 
      @click="emit('openAuthorization')"
      class="flex items-center gap-3 text-gray-500 hover:text-black cursor-pointer">
        <img src="/profile.svg" alt="profile" />
        <span>Профиль</span>
      </li>
    </ul>
  </header>
</template>
