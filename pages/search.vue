<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import AdList from '~/components/AdList.vue'

const router = useRouter()

const allAds = ref([])
const maxPrice = ref(null)
const selectedRegion = ref('')
const selectedRooms = ref(null)
const onlyFree = ref(false)
const contactAd = ref(null)
const selectedConditions = ref([])

const conditionsOptions = [
  'Без дітей',
  'Без тварин',
  'Для сімей з дітьми',
  'Для людей з інвалідністю',
  'Можна з тваринами',
  'Для одиноких людей',
  'Тимчасове проживання',
  'Довготривале проживання',
]

const regions = [
  'Вінницька', 'Волинська', 'Дніпропетровська', 'Донецька',
  'Житомирська', 'Закарпатська', 'Запорізька', 'Івано-Франківська',
  'Київська', 'Кіровоградська', 'Луганська', 'Львівська',
  'Миколаївська', 'Одеська', 'Полтавська', 'Рівненська',
  'Сумська', 'Тернопільська', 'Харківська', 'Херсонська',
  'Хмельницька', 'Черкаська', 'Чернівецька', 'Чернігівська', 'Київ'
]

onMounted(async () => {
  try {
    const response = await fetch('/ads.json')
    if (!response.ok) throw new Error('Failed to load ads')
    const data = await response.json()
    console.log('Fetched ads:', data)   
    allAds.value = data
  } catch (error) {
    console.error('Error loading ads:', error)
  }
})

const filteredAds = computed(() => {
  return allAds.value.filter(ad => {
    if (onlyFree.value && ad.price > 0) return false
    if (maxPrice.value !== null && ad.price > maxPrice.value) return false
    if (selectedRegion.value && ad.region !== selectedRegion.value) return false
    if (selectedRooms.value && ad.rooms !== selectedRooms.value) return false
    if (selectedConditions.value.length > 0) {
      if (!Array.isArray(ad.conditions) || !selectedConditions.value.every(cond => ad.conditions.includes(cond))) {
        return false
      }
    }
    return true
  })
})

function contact(ad) {
  contactAd.value = ad
}

const goBack = () => {
  router.go(-1)
}
</script>


<template>
<div v-for="ad in allAds" :key="ad.id">
  <p>{{ ad.title }} — {{ ad.price }}</p>
</div>
</template>

<style>
.price-input{
    width: 180px;
    height: 34px;
    border-radius: 4px;
    border: none;
    outline: none;
    text-indent: 4px;
}

select{
    width: 180px;
    height: 34px;
    border-radius: 4px;
    border: none;
    outline: none;
    text-indent: 4px;
}
.label{
    margin-right: 10px;
    font-weight: 600;
}

.sigment-wrapper{
    margin-bottom: 20px;
    
}
</style>