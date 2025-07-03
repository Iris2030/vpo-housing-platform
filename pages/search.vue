<script setup>
import { ref, computed, onMounted } from 'vue'
import { ads as initialAds } from '~/data/ads.js'
import AdList from '~/components/AdList'

const maxPrice = ref(null)
const selectedRegion = ref('')
const selectedRooms = ref(null)
const contactAd = ref(null)

const regions = ['Київ', 'Львів', 'Одеська']

const allAds = ref([])

onMounted(() => {
    const saved = JSON.parse(localStorage.getItem('ads') || '[]')
    // Обʼєднати з initialAds
    allAds.value = [...initialAds, ...saved]
})

const filteredAds = computed(() => {
    return allAds.value.filter(ad => {
        if (maxPrice.value && ad.price > maxPrice.value) return false
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
            <input type="number" v-model.number="maxPrice" placeholder="Максимальна ціна" />
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
