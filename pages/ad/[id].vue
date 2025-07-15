<script setup>
import { onMounted, ref } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const ad = ref(null)

const selectedAd = ref(null)
const showModal = ref(false)

function handleContact(ad) {
  selectedAd.value = ad
  showModal.value = true
}

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
    <button class="accept-btn" @click="handleContact(ad)">Зв’язатися</button>
    <br>
    <NuxtLink class="back" to="/">← Назад</NuxtLink>
  </div>
  <div v-else>
    <p>Оголошення не знайдено.</p>
    <NuxtLink to="/">← Повернутись на головну</NuxtLink>
  </div>
  
  <!-- Модальне вікно -->
<div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
    <div class="modal">
      <p><strong>Номер телефону:</strong></p>
      <a :href="`tel:${ad.phone }`" class="phone-link">
        📞 {{ ad.phone  }}
      </a>
      <button @click="showModal = false" class="accept-btn" style="margin-top: 10px;">Закрити</button>
    </div>
  </div>
  
</template>

<style scoped>
button{
  margin-bottom: 8px;
  width: 240px;
}


.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.4);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal {
  background: white;
  padding: 20px;
  border-radius: 8px;
  width: 300px;
  text-align: center;
}

.phone-link {
  display: inline-block;
  font-size: 18px;
  font-weight: bold;
  margin-top: 10px;
  color: #2a73dd;
  text-decoration: none;
}


</style>