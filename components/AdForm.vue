<script setup>
import { reactive, ref, watch } from 'vue'

const errors = reactive({
  title: '',
  price: '',
  phone: '',
  region: '',
  city: '',
  address: '',
  rooms: ''
})

const message = ref('')

function clearErrors() {
  for (const key in errors) {
    errors[key] = ''
  }
}

const emit = defineEmits(['submitted'])

const regions = [
"Вінницька", "Волинська", "Дніпропетровська", "Донецька", "Житомирська",
"Закарпатська", "Запорізька", "Івано-Франківська", "Київська", "Кіровоградська",
"Луганська", "Львівська", "Миколаївська", "Одеська", "Полтавська",
"Рівненська", "Сумська", "Тернопільська", "Харківська", "Херсонська",
"Хмельницька", "Черкаська", "Чернівецька", "Чернігівська"
]

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

function handlePhoneInput(e) {
  if (!form.phone.startsWith('+380')) {
    form.phone = '+380' + form.phone.replace(/^\+?380?/, '').replace(/\D/g, '').slice(0, 9)
  } else {
    form.phone = '+380' + form.phone.replace(/^\+?380?/, '').replace(/\D/g, '').slice(0, 9)
  }
}

function isValidPhone(phone) {
  return /^\+380\d{9}$/.test(phone)
}


function submitForm() {
  clearErrors()

  let hasError = false

  if (!form.title || form.title.length < 5) {
    errors.title = 'Заголовок має бути не менше 5 символів.'
    hasError = true
  }

  if (form.price === null || form.price < 0) {
    errors.price = 'Ціна має бути 0 або більше.'
    hasError = true
  }

  if (!form.region) {
    errors.region = 'Оберіть регіон.'
    hasError = true
  }

  if (!form.city) {
    errors.city = 'Вкажіть місто.'
    hasError = true
  }

  if (!form.address) {
    errors.address = 'Вкажіть адресу.'
    hasError = true
  }

  if (!form.rooms) {
    errors.rooms = 'Оберіть кількість кімнат.'
    hasError = true
  }

  if (!isValidPhone(form.phone)) {
    errors.phone = 'Телефон має бути у форматі +380XXXXXXXXX.'
    hasError = true
  }

  if (hasError) return

  const newAd = {
    id: Date.now(),
    ...form
  }

  const existing = JSON.parse(localStorage.getItem('ads') || '[]')
  existing.push(newAd)
  localStorage.setItem('ads', JSON.stringify(existing))

  emit('submitted', newAd)

  message.value = 'Оголошення додано успішно!'

  clearErrors()

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

watch(() => props.selectedCoords, (newCoords) => {
  if (newCoords) {
    form.lat = newCoords.lat
    form.lng = newCoords.lng
    fetchAddress(newCoords.lat, newCoords.lng)
  }
})
</script>


<template>
  <form @submit.prevent="submitForm">
    <p v-if="form.lat && form.lng">
      Координати вибрано: {{ form.lat.toFixed(4) }}, {{ form.lng.toFixed(4) }}
    </p>
    
    <div class="section">
      <label>Заголовок</label><br />
      <p v-if="errors.title" style="color: red; margin-bottom: 4px;">{{ errors.title }}</p>
      <input class="form-input" v-model="form.title" />
    </div>
    
    <div class="section">
      <label>Ціна (0 — якщо безкоштовно)</label><br />
      <p v-if="errors.price" style="color: red; margin-bottom: 4px;">{{ errors.price }}</p>
      <input class="form-input" type="number" min="0" v-model.number="form.price" />
    </div>
    
    <div class="section">
      <label>Регіон</label><br />
      <p v-if="errors.region" style="color: red; margin-bottom: 4px;">{{ errors.region }}</p>
      <select v-model="form.region">
        <option disabled value="">Оберіть регіон</option>
        <option v-for="r in regions" :key="r" :value="r">{{ r }}</option>
      </select>
    </div>
    
    <div class="section">
      <label>Місто</label><br />
      <p v-if="errors.city" style="color: red; margin-bottom: 4px;">{{ errors.city }}</p>
      <input class="form-input" v-model="form.city" placeholder="Наприклад, Київ" />
    </div>
    
    <div class="section">
      <label>Адреса</label><br />
      <p v-if="errors.address" style="color: red; margin-bottom: 4px;">{{ errors.address }}</p>
      <input class="form-input" v-model="form.address" />
    </div>
    
    <div class="section">
      <label>Телефон</label><br />
      <p v-if="errors.phone" style="color: red; margin-bottom: 4px;">{{ errors.phone }}</p>
      <input
      class="form-input"
      v-model="form.phone"
      @input="handlePhoneInput"
      placeholder="+380XXXXXXXXX"
      />
    </div>
    
    <div class="section">
      <label>Кількість кімнат</label><br />
      <p v-if="errors.rooms" style="color: red; margin-bottom: 4px;">{{ errors.rooms }}</p>
      <select v-model.number="form.rooms">
        <option disabled :value="null">Оберіть кількість</option>
        <option :value="1">1</option>
        <option :value="2">2</option>
        <option :value="3">3</option>
        <option :value="4">4+</option>
      </select>
    </div>
    
    <button type="submit">Додати</button>
    <p v-if="message" style="color: green; margin-top: 10px;">{{ message }}</p>
    <!-- <p v-if="message" style="color: red;">{{ message }}</p> -->
  </form>
</template>


<style scoped>
input{
  margin-top: 6px;
  width: 240px;
  height: 34px;
  border-radius: 4px;
  border: none;
  outline: none;
  text-indent: 4px;
}

select{
  margin-top: 6px;
  width: 240px;
}

button{
  margin-top: 20px;
  width: 240px;
  height: 34px;
  background-color: #000000;
  border-radius: 4px;
  border: 1px solid rgb(170, 169, 169);
  color: white;
}
label{
  margin-bottom: 6px;
}

.section{
  margin-top: 10px;
}
</style>