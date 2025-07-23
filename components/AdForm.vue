<script setup>
import { reactive, ref, watch } from 'vue'

const errors = reactive({
  title: '',
  price: '',
  phone: '',
  region: '',
  city: '',
  address: '',
  rooms: '',
  description: '',
  conditions: ''
})
const conditions = ref([])
const message = ref('')
const conditionsList = [
"Без дітей",
"Без тварин",
"Для сімей з дітьми",
"Для людей з інвалідністю",
"Можна з тваринами",
"Для одиноких людей",
"Тимчасове проживання",
"Довготривале проживання"
]

function toggleCondition(condition) {
  const index = form.conditions.indexOf(condition)
  if (index === -1) {
    form.conditions.push(condition)
  } else {
    form.conditions.splice(index, 1)
  }
}

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
  lng: null,
  description: '',
  conditions: []
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
  
  if (!form.description) {
    errors.description = 'Додайте опис житла.'
    hasError = true
  }
  
  if (!form.conditions.length) {
    errors.conditions = 'Оберіть хоча б одну умову.'
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
    lng: null,
    description: '',
    conditions: []
  })
}

watch(() => props.selectedCoords, (newCoords) => {
  if (newCoords) {
    form.lat = newCoords.lat
    form.lng = newCoords.lng
    fetchAddress(newCoords.lat, newCoords.lng)
  }
})

watch(() => form.conditions, (newVal) => {
  console.log('Selected conditions:', newVal)
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
    
    <div class="section">
      <label>Опис</label><br />
      <p v-if="errors.description" style="color: red; margin-bottom: 4px;">{{ errors.description }}</p>
      <textarea v-model="form.description" placeholder="Простора квартира в центрі..."></textarea>
    </div>
    
    <div class="section">
      <label>Умови проживання</label><br />
      <p v-if="errors.conditions" style="color: red; margin-bottom: 4px;">{{ errors.conditions }}</p>
      <div v-for="condition in conditionsList" :key="condition">
        <label class="checkbox">
          <input
          type="checkbox"
          :value="condition"
          :checked="form.conditions.includes(condition)"
          @change="toggleCondition(condition)"
          />
          {{ condition }}
        </label>
      </div>
      
      <ul class="conditions-list">
        <li v-for="(condition, index) in form.conditions" :key="index" class="conditions-item">{{ condition }}</li>
      </ul>
    </div>
    
    <button type="submit">Додати</button>
    <p v-if="message" style="color: green; margin-top: 10px;">{{ message }}</p>
    <!-- <p v-if="message" style="color: red;">{{ message }}</p> -->
  </form>
</template>


<style scoped>

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
  cursor: pointer;
}

button:hover{
  color: white;
  background-color:  #3e3e3e;
}
label{
  margin-bottom: 6px;
}

textarea{
  margin-top: 6px;
  width: 240px;
  height: 160px;
  border-radius: 4px;
  border: none;
  outline: none;
  text-indent: 4px;
  resize: none;
}

.section{
  margin-top: 10px;
}

.conditions-item{
  width: auto;
  border: solid 1px #000;
  border-radius: 4px;
  padding: 4px;
  background-color: #fff;
}

.conditions-item:not(:last-child){
  margin-right: 12px;
  
}

.conditions-list{
  padding-left: 0;
  display: flex;
}
</style>