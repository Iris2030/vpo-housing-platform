<script setup>
import { ref, computed, onMounted } from 'vue'
import { ads as initialAds } from '~/data/ads.js'
import AdList from '~/components/AdList.vue'

const maxPrice = ref(null)
const selectedRegion = ref('')
const selectedRooms = ref(null)
const onlyFree = ref(false) // новий фільтр
const contactAd = ref(null)

const regions = [
  'Вінницька',
  'Волинська',
  'Дніпропетровська',
  'Донецька',
  'Житомирська',
  'Закарпатська',
  'Запорізька',
  'Івано-Франківська',
  'Київська',
  'Кіровоградська',
  'Луганська',
  'Львівська',
  'Миколаївська',
  'Одеська',
  'Полтавська',
  'Рівненська',
  'Сумська',
  'Тернопільська',
  'Харківська',
  'Херсонська',
  'Хмельницька',
  'Черкаська',
  'Чернівецька',
  'Чернігівська',
  'Київ' 
]


const allAds = ref([])

onMounted(() => {
  const saved = JSON.parse(localStorage.getItem('ads') || '[]')
  allAds.value = [...initialAds, ...saved]
})

const filteredAds = computed(() => {
  return allAds.value.filter(ad => {
    if (onlyFree.value && ad.price > 0) return false
    if (maxPrice.value !== null && ad.price > maxPrice.value) return false
    if (selectedRegion.value && ad.region !== selectedRegion.value) return false
    if (selectedRooms.value && ad.rooms !== selectedRooms.value) return false
    return true
  })
})

function contact(ad) {
  contactAd.value = ad
}
</script>

<template>
  <div>
    <h2>Пошук житла</h2>

    <div>
      <label>Ціна до:</label>
      <input type="number" min="0" v-model.number="maxPrice" placeholder="Максимальна ціна" />
    </div>

    <div>
      <label>Регіон:</label>
      <select v-model="selectedRegion">
        <option value="">Всі</option>
        <option v-for="region in regions" :key="region" :value="region">{{ region }}</option>
      </select>
    </div>

    <div>
      <label>Кількість кімнат:</label>
      <select v-model.number="selectedRooms">
        <option :value="null">Всі</option>
        <option :value="1">1</option>
        <option :value="2">2</option>
        <option :value="3">3</option>
        <option :value="4">4</option>
      </select>
    </div>

    <div>
      <label>
        <input type="checkbox" v-model="onlyFree" />
        Лише безкоштовне житло
      </label>
    </div>

    <!-- Список оголошень -->
    <AdList :ads="filteredAds" @contact="contact" />
    <p v-if="filteredAds.length === 0" style="margin-top: 20px;">
      Оголошень не знайдено за заданими фільтрами.
    </p>

    <div v-if="contactAd" style="margin-top: 20px; padding: 10px; background: #eee;">
      <h4>Контакт для: {{ contactAd.title }}</h4>
      <p>Телефон: {{ contactAd.phone }}</p>
      <button @click="contactAd = null">Закрити</button>
    </div>
  </div>
</template>
