<script setup>
import MapView from '~/components/MapView.vue'
import { ads as initialAds } from '~/data/ads.js'
import { ref, onMounted, computed } from 'vue'

const categories = [
{ key: 'schools', label: '🏫 Школи' },
{ key: 'churches', label: '⛪ Церкви' },
{ key: 'shelters', label: '🛡️ Укриття' },
{ key: 'hospitals', label: '🏥 Лікарні' },
{ key: 'bus_stops', label: '🚌 Зупинки' },
{ key: 'supermarkets', label: '🛒 Магазини' },
]

const allAds = ref([])
const activeLayers = ref({})
categories.forEach(cat => {
  activeLayers.value[cat.key] = false
})

const selectedRooms = ref(null)
const onlyFree = ref(false)

onMounted(() => {
  const saved = JSON.parse(localStorage.getItem('ads') || '[]')
  allAds.value = [...initialAds, ...saved]
})

const filteredAds = computed(() => {
  return allAds.value.filter(ad => {
    if (onlyFree.value && ad.price > 0) return false
    if (selectedRooms.value !== null) {
      if (selectedRooms.value === 4) {
        if (ad.rooms < 4) return false
      } else {
        if (ad.rooms !== selectedRooms.value) return false
      }
    }
    return true
  })
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
        <p>Фільтри</p>
        <label class="checkbox" v-for="cat in categories" :key="cat.key">
          <input type="checkbox" v-model="activeLayers[cat.key]" />
          {{ cat.label }}
        </label>
        
        <div>
          <p>Житло</p>
          <label >Кількість кімнат:</label>
          <select class="rooms-select"  v-model.number="selectedRooms">
            <option :value="null">Всі</option>
            <option :value="1">1</option>
            <option :value="2">2</option>
            <option :value="3">3</option>
            <option :value="4">4+</option>
          </select>
        </div>
        
        <div>
          <label class="checkbox">
            <input type="checkbox" v-model="onlyFree" />
            Безкоштовне
          </label>
        </div>
      </div>
      
      
      <MapView
      :ads="filteredAds"
      :layers="activeLayers"
      :filters="{ rooms: selectedRooms, onlyFree }"
      />
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
  min-width: 180px;
}

.rooms-select{
  margin-top: 8px;
}

@media screen and (max-width: 460px) {
  
  h1{
    font-size: 24px;
  }
  
  .filters-wrapper{
    display: grid;
  }
}
</style>
