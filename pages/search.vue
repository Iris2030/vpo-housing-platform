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
    <div>
        <button class="back" @click="goBack">← Назад</button>
        <h2>Пошук житла</h2>
        
        <div class="sigment-wrapper">
            <label class="label">Ціна до:</label>
            <input class="price-input" type="number" min="0" v-model.number="maxPrice" placeholder="Максимальна ціна" />
        </div>
        
        <div class="sigment-wrapper">
            <label class="label">Регіон:</label>
            <select v-model="selectedRegion">
                <option value="">Всі</option>
                <option v-for="region in regions" :key="region" :value="region">{{ region }}</option>
            </select>
        </div>
        
        <div class="sigment-wrapper">
            <label class="label">Кількість кімнат:</label>
            <select v-model.number="selectedRooms">
                <option :value="null">Всі</option>
                <option :value="1">1</option>
                <option :value="2">2</option>
                <option :value="3">3</option>
                <option :value="4">4</option>
            </select>
        </div>
        
        <div class="sigment-wrapper">
            <label class="label checkbox">
                <input type="checkbox" v-model="onlyFree" />
                Лише безкоштовне житло
            </label>
        </div>
        
        <div class="sigment-wrapper">
            <label class="label">Умови проживання:</label>
            <div v-for="cond in conditionsOptions" :key="cond" class="checkbox-wrapper">
                <label class="checkbox">
                    <input
                    type="checkbox"
                    :value="cond"
                    v-model="selectedConditions"
                    />
                    {{ cond }}
                </label>
            </div>
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