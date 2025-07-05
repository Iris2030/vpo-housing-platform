<script setup>
defineProps({
  ads: Array
})

defineEmits(['contact'])
</script>

<template>
  <ul class="list">
    <li
    v-for="ad in ads"
    :key="ad.id"
    style="margin-bottom: 15px; background: white; padding: 10px; border-radius: 5px;"
    >
    <h3>{{ ad.title }}</h3>
    <p><span class="list-text"> Адреса:</span> {{ ad.address }}</p>
    <p><span class="list-text">Ціна:</span>  {{ ad.price }} грн</p>
    <p><span class="list-text">Регіон:</span>  {{ ad.region }}</p>
    <p><span class="list-text">Кімнат:</span>  {{ ad.rooms }}</p>
    <button class="accept-btn" @click="$emit('contact', ad)">Зв’язатися</button>
  </li>
</ul>
</template>

<style>
.list{
  list-style: none;
  padding: 0;
}

.list-text{
    font-weight: 600;
}

.accept-btn{
  width: 200px;
  height: 34px;
  background-color: #fff;
  border-radius: 4px;
  border: 1px solid rgb(170, 169, 169);
}
</style>