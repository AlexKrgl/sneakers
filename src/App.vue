<script setup>
import { ref, reactive, provide, computed, watch } from 'vue'
import axios from 'axios'
import Header from './components/Header.vue'
import Drawer from './components/Drawer.vue'


/* (start) Корзина... */
const cart = ref([]);

const drawerOpen = ref(false);


const totalPrice = computed( // применяем, когда нам нужно использовать вычисления в других местах (компонентах)
  () => cart.value.reduce((acc, item) => acc + item.price, 0)
);
const vatPrice = computed(() => Math.round(totalPrice.value * 5) / 100)


const closeDrawer = () => {
  drawerOpen.value = false;
}

const openDrawer = () => {
  drawerOpen.value = true;
}

const addToCart = (item) => {
  cart.value.push(item);
    item.isAdded = true;
}

const removeFromCart = (item) => {
  cart.value.splice(cart.value.indexOf(item), 1);
    item.isAdded = false;
}

watch(cart, () => {
  localStorage.setItem('cart', JSON.stringify(cart.value))
},
{ deep: true}
)

provide('cart', {
  cart,
  addToCart,
  removeFromCart,
  closeDrawer,
  openDrawer
})

/* ...Корзина (end) */

// Авторизация
const isRegistr = ref(false);
const authorizationOpen = ref(false);

const openAuthorization = () => {
  authorizationOpen.value = true;
}

const closeAuthorization = () => {
  authorizationOpen.value = false;
}

const submitRegistration = async () =>{
    try {
        const { data } = await axios.get('https://d5e1d70c5e113cd5.mokky.dev/users');
        
        for(let user of data) {
            if (user.userMail == newUser.userMail) {
               return alert('Такой пользователь уже есть');
            }
        }
        await axios.post('https://d5e1d70c5e113cd5.mokky.dev/users', newUser);
        alert('Вы успешно зарегистрировались');
        closeAuthorization;
    } catch (err) {
        console.log(err)
    }
}

const submitAuthorization = async () =>{
    try {
        const { data } = await axios.get('https://d5e1d70c5e113cd5.mokky.dev/users');
        
        for(let user of data) {
            if (user.userMail == userAuth.mailUser) {
                if (user.userPassword == userAuth.passwordUser) {
                    alert('Вы успешно авторизировались');
                    closeAuthorization;
                    return user
                } else {
                    alert('Пароль не совпадает')
                }
               return alert('Такой пользователь уже есть');
            }
        }
        return alert('Данные введены неверно');
    } catch (err) {
        console.log(err)
    }
}

const newUser = reactive({
    userName: '',
    userMail: '',
    userPhone: '',
    userPassword: ''
});

const userAuth = reactive({
    mailUser: '',
    passwordUser: ''
})

const offRegistrBtn = () => {
    isRegistr.value = false
}

const onRegistrBtn = () => {
    isRegistr.value = true
}



</script>

<template>
  <Authorization 
  v-if="authorizationOpen">
  <div class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70"></div>

    <div class="fixed top-0 left-0 z-20 h-full w-full flex items-center justify-center">
        <div class="bg-lime-200 relative w-96 h-autho p-8 p-10 ">
            <img @click="closeAuthorization" class="absolute right-1 top-1 cursor-pointer" src="/close.svg" alt="X">
            <div class="flex justify-center gap-2 mb-5">
                <button @click="offRegistrBtn" class="bg-stone-300 border-solid border-2 w-1/2 p-1">Авторизация</button>
                <button @click="onRegistrBtn" class="bg-stone-300 border-solid border-2 w-1/2 p-1">Регистрация</button>
            </div>
            <!-- @submit.prevent.stop - отменяет стандартное поведение браузера -->
            <form v-if="isRegistr" novalidate @submit.prevent.stop="submitRegistration" class="flex flex-col justify-center gap-2" action=""> 
                <input v-model="newUser.userName" class="w-full p-1 " type="text" placeholder="Введите имя">
                <input v-model="newUser.userMail" class="w-full p-1 " type="text" placeholder="Введите почту">
                <input v-model="newUser.userPhone" class="w-full p-1 " type="text" placeholder="Введите телефон">
                <input v-model="newUser.userPassword" class="w-full p-1 " type="text" placeholder="Введите пароль">
                <button type="submit" class="bg-stone-300 border-solid border-2 w-full p-1 hover:bg-slate-600">Зарегистрироваться</button>
            </form>

            <form v-if="!isRegistr" novalidate @submit.prevent.stop="submitAuthorization" class="flex flex-col justify-center gap-2" action=""> 
                <input v-model="userAuth.mailUser" class="w-full p-1 " type="text" placeholder="Введите почту">
                <input v-model="userAuth.passwordUser" class="w-full p-1 " type="text" placeholder="Введите пароль">
                <button type="submit" class="bg-stone-300 border-solid border-2 w-full p-1 hover:bg-slate-600">Зарегистрироваться</button>
            </form>
        </div>
    </div>
  </Authorization>
  <Drawer 
  v-if="drawerOpen" 
  :total-price="totalPrice" 
  :vat-price="vatPrice" />

  <div class="bg-white w-4/5 m-auto rounded-xl shadow-xl mt-10">
    <Header :total-price="totalPrice" @open-drawer="openDrawer"  @open-authorization="openAuthorization" />

    <div class="p-10">
      <router-view></router-view>
    </div>
  </div>
</template>
