<script setup>
import { onMounted, ref } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const ad = ref(null)

onMounted(async () => {
  const allAds = JSON.parse(localStorage.getItem('ads') || '[]')

  try {
    const { ads: initialAds } = await import('~/data/ads.js')
    allAds.push(...initialAds)
  } catch (e) {
    console.warn('initialAds не знайдено або не підключено')
  }

  const id = Number(route.params.id)
  ad.value = allAds.find(a => a.id === id)
})
</script>


<template>
  <div v-if="ad">
    <h2>{{ ad.title }}</h2>
    <p><strong>Ціна:</strong> {{ ad.price === 0 ? 'Безкоштовно' : ad.price + ' грн' }}</p>
    <p><strong>Кількість кімнат:</strong> {{ ad.rooms }}</p>
    <p><strong>Регіон:</strong> {{ ad.region }}</p>
    <p><strong>Місто:</strong> {{ ad.city }}</p>
    <p><strong>Адреса:</strong> {{ ad.address }}</p>
    <p><strong>Телефон:</strong> {{ ad.phone }}</p>
    <p><strong>Координати:</strong> {{ ad.lat.toFixed(4) }}, {{ ad.lng.toFixed(4) }}</p>
    <NuxtLink to="/">← Назад</NuxtLink>
  </div>
  <div v-else>
    <p>Оголошення не знайдено.</p>
    <NuxtLink to="/">← Повернутись на головну</NuxtLink>
  </div>
</template>
