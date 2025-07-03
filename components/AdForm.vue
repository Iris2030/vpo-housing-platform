<script setup>
import { reactive, ref, watch } from 'vue'

const emit = defineEmits(['submitted'])

const regions = [
  "Вінницька", "Волинська", "Дніпропетровська", "Донецька", "Житомирська",
  "Закарпатська", "Запорізька", "Івано-Франківська", "Київська", "Кіровоградська",
  "Луганська", "Львів", "Миколаїв", "Одеська", "Полтавська",
  "Рівненська", "Сумська", "Тернопільська", "Харківська", "Херсонська",
  "Хмельницька", "Черкаська", "Чернівецька", "Чернігівська"
]
const message = ref('')

const props = defineProps({
  selectedCoords: Object
})

const form = reactive({
  title: '',
  price: null,
  region: '',
  city: '',
  address: '',
  phone: '',
  rooms: null,
  lat: null,
  lng: null
})

async function fetchAddress(lat, lng) {
  try {
    const url = `https://nominatim.openstreetmap.org/reverse?format=json&lat=${lat}&lon=${lng}&accept-language=uk`
    const res = await fetch(url)
    if (!res.ok) throw new Error('Помилка при отриманні адреси')
    const data = await res.json()

    if (data.address) {
      form.address = data.display_name || ''
      form.city = data.address.city || data.address.town || data.address.village || ''

      const rawState = data.address.state || ''
      const normalizedState = rawState.replace(' область', '').trim()

      if (regions.includes(normalizedState)) {
        form.region = normalizedState
      }
    }
  } catch (e) {
    console.error('Помилка геокодування:', e)
  }
}

watch(() => props.selectedCoords, (newCoords) => {
  if (newCoords) {
    form.lat = newCoords.lat
    form.lng = newCoords.lng
    fetchAddress(newCoords.lat, newCoords.lng)
  }
})

function submitForm() {
  if (!form.title || form.price === null || !form.region || !form.city || !form.address || !form.phone || !form.rooms) {
    message.value = 'Будь ласка, заповніть всі поля.'
    return
  }

  const newAd = {
    id: Date.now(),
    ...form
  }

  const existing = JSON.parse(localStorage.getItem('ads') || '[]')
  existing.push(newAd)
  localStorage.setItem('ads', JSON.stringify(existing))

  emit('submitted', newAd)
  message.value = 'Оголошення додано!'

  Object.assign(form, {
    title: '',
    price: null,
    region: '',
    city: '',
    address: '',
    phone: '',
    rooms: null,
    lat: null,
    lng: null
  })
}
</script>


<template>
  <form @submit.prevent="submitForm">
    <p v-if="form.lat && form.lng">
      Координати вибрано: {{ form.lat.toFixed(4) }}, {{ form.lng.toFixed(4) }}
    </p>

    <div>
      <label>Заголовок</label><br />
      <input v-model="form.title" />
    </div>

    <div>
      <label>Ціна (0 — якщо безкоштовно)</label><br />
      <input type="number" min="0" v-model.number="form.price" />
    </div>

    <div>
      <label>Регіон</label><br />
      <select v-model="form.region">
        <option disabled value="">Оберіть регіон</option>
        <option v-for="r in regions" :key="r" :value="r">{{ r }}</option>
      </select>
    </div>

    <div>
      <label>Місто</label><br />
      <input v-model="form.city" placeholder="Наприклад, Київ" />
    </div>

    <div>
      <label>Адреса</label><br />
      <input v-model="form.address" />
    </div>

    <div>
      <label>Телефон</label><br />
      <input v-model="form.phone" />
    </div>

    <div>
      <label>Кількість кімнат</label><br />
      <select v-model.number="form.rooms">
        <option disabled :value="null">Оберіть кількість</option>
        <option :value="1">1</option>
        <option :value="2">2</option>
        <option :value="3">3</option>
        <option :value="4">4+</option>
      </select>
    </div>

    <button type="submit">Додати</button>
    <p v-if="message" style="color: green;">{{ message }}</p>
  </form>
</template>


