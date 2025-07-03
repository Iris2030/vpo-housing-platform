<script setup>
import MapView from '~/components/MapView.vue'
import { ads as initialAds } from '~/data/ads.js'
import { ref, onMounted } from 'vue'

const categories = [
  { key: 'schools', label: '🏫 Школи' },
  { key: 'churches', label: '⛪ Церкви' },
  { key: 'shelters', label: '🛡️ Укриття' },
  { key: 'hospitals', label: '🏥 Лікарні' },
  { key: 'bus_stops', label: '🚌 Зупинки транспорту' },
  { key: 'supermarkets', label: '🛒 Супермаркети' },
  // { key: 'museums', label: '🏛️ Музеї' }

]

const allAds = ref([])
const activeLayers = ref({})
categories.forEach(cat => {
  activeLayers.value[cat.key] = false
})

onMounted(() => {
  const saved = JSON.parse(localStorage.getItem('ads') || '[]')
  allAds.value = [...initialAds, ...saved]
})
</script>

<template>
  <div>
    <h1>Ласкаво просимо на платформу пошуку житла для ВПО</h1>
    <p>Проста демонстрація концепту сервісу.</p>
    <NuxtLink to="/search">Знайти житло</NuxtLink> |
    <NuxtLink to="/submit">Додати оголошення</NuxtLink>

    <div class="filters-wrapper">
      <div class="filters">
        <label v-for="cat in categories" :key="cat.key">
          <input type="checkbox" v-model="activeLayers[cat.key]" />
          {{ cat.label }}
        </label>
      </div>

      <MapView :ads="allAds" :layers="activeLayers" />
    </div>
  </div>
</template>

<style>
.name-wrapper {
  display: flex;
  align-items: center;
}
.name {
  font-size: 20px;
  margin-left: 8px;
}
.filters-wrapper {
  display: flex;
  gap: 20px;
  margin-top: 20px;
}
.filters {
  display: flex;
  flex-direction: column;
  min-width: 150px;
}
</style>
