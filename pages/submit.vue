<script setup>
import AdForm from '~/components/AdForm.vue'
import MapView from '~/components/MapView.vue'
import { ref } from 'vue'

const selectedCoords = ref(null)

function onMapClick(coords) {
  selectedCoords.value = coords
}

function onSubmit(ad) {
  console.log('Оголошення збережено:', ad)
}
</script>

<template>
    <div>
      <h2>Додати оголошення</h2>
      <AdForm :selected-coords="selectedCoords" @submitted="onSubmit" />
      <MapView :ads="[]" interactive @mapClick="onMapClick" />
    </div>
  </template>
