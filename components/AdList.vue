<script setup>
import { ref } from 'vue'

const selectedAd = ref(null)
const showModal = ref(false)

function handleContact(ad) {
  selectedAd.value = ad
  showModal.value = true
}

defineProps({
  ads: Array
})

</script>

<template>
  <ul class="list">
    <li
    v-for="ad in ads"
    :key="ad.id"
    style="margin-bottom: 15px; background: white; padding: 10px; border-radius: 5px;"
    >
    <h3>{{ ad.title }}</h3>
    <p><span class="list-text"> Адреса:</span> {{ ad.address }}</p>
    <p><span class="list-text">Ціна:</span>  {{ ad.price }} грн</p>
    <p><span class="list-text">Регіон:</span>  {{ ad.region }}</p>
    <p><span class="list-text">Кімнат:</span>  {{ ad.rooms }}</p>
    <button class="accept-btn" @click="handleContact(ad)">Зв’язатися</button>
  </li>
</ul>

<!-- Модальне вікно -->
<div v-if="showModal" class="modal-overlay" @click.self="showModal = false">
  <div class="modal">
    <p><strong>Номер телефону:</strong></p>
    <a :href="`tel:${selectedAd.phone}`" class="phone-link">
      📞 {{ selectedAd.phone }}
    </a>
    <button @click="showModal = false" class="accept-btn" style="margin-top: 10px;">Закрити</button>
  </div>
</div>
</template>

<style>
.list{
  list-style: none;
  padding: 0;
}

.list-text{
  font-weight: 600;
}

.accept-btn{
  width: 200px;
  height: 34px;
  background-color: #fff;
  border-radius: 4px;
  border: 1px solid rgb(170, 169, 169);
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