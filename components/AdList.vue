<script setup>
import { ref, onMounted, watch } from 'vue'
import { useRouter } from 'vue-router'

const props = defineProps({
  ads: {
    type: Array
  }
})
const emit = defineEmits(['contact'])
const router = useRouter()

const selectedAd = ref(null)
const showModal = ref(false)

function handleContact(ad) {
  selectedAd.value = ad
  showModal.value = true
}

function goToAd(ad) {
  router.push(`/ad/${ad.id}`)
}

</script>

<template>
  <div class="ad-list" v-if="ads && ads.length">
    <div
    v-for="ad in ads"
    :key="ad.id"
    class="ad-card"
    @click="goToAd(ad)"
    style="cursor: pointer; border: 1px solid #ccc; padding: 10px; margin-bottom: 10px;"
    >
    <h3>{{ ad.title }}</h3>
    <p>Ціна: {{ ad.price === 0 ? 'Безкоштовно' : ad.price + ' грн' }}</p>
    <p>Область: {{ ad.region }}</p>
    <p>Кімнат: {{ ad.rooms }}</p>
    <p>Опис: {{ ad.description }} </p>
    <p>Умови проживання:</p>
    <ul class="conditions-list">
      <li v-for="(condition, index) in ad.conditions" :key="index" class="conditions-item">{{ condition }}</li>
    </ul>
  </div>
</div>
</template>

<style>

.ad-list{
height: auto;
}

.list{
  list-style: none;
  padding: 0;
}

.list-text{
  font-weight: 600;
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

.ad-list div{
  background-color: #fff;
}
</style>