<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import AdForm from '~/components/AdForm.vue'
import MapView from '~/components/MapView.vue'

const router = useRouter()
const selectedCoords = ref(null)

function onMapClick(coords) {
  selectedCoords.value = coords
}

function onSubmit(ad) {
  console.log('Оголошення збережено:', ad)
}

const goBack = () => {
  router.go(-1)
}
</script>

<template>
    <div>
      <button class="back" @click="goBack">← Назад</button>
      <h2>Додати оголошення</h2>
      <AdForm :selected-coords="selectedCoords" @submitted="onSubmit" />
      <MapView :ads="[]" interactive @mapClick="onMapClick" />
    </div>
  </template>
