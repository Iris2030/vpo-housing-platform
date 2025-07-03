<script setup>
defineProps({
  ads: Array
})

defineEmits(['contact'])
</script>

<template>
    <ul>
      <li
        v-for="ad in ads"
        :key="ad.id"
        style="margin-bottom: 15px; background: white; padding: 10px; border-radius: 5px;"
      >
        <h3>{{ ad.title }}</h3>
        <p>Ціна: {{ ad.price }} грн</p>
        <p>Адреса: {{ ad.address }}</p>
        <p>Регіон: {{ ad.region }}</p>
        <p>Кімнат: {{ ad.rooms }}</p>
        <button @click="$emit('contact', ad)">Зв’язатися</button>
      </li>
    </ul>
  </template>
  
  