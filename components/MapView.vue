<script setup>
import { onMounted, watch, ref } from 'vue'

const props = defineProps({
  ads: Array,
  layers: Object, // наприклад { schools: true, churches: false, shelters: true }
  interactive: Boolean
})

const emit = defineEmits(['mapClick'])

const map = ref(null)
const selectedMarker = ref(null)
const extraMarkers = ref([])

function getBBox() {
  if (!map.value) return null
  const bounds = map.value.getBounds()
  return {
    south: bounds.getSouth(),
    west: bounds.getWest(),
    north: bounds.getNorth(),
    east: bounds.getEast()
  }
}

async function fetchPOI(type, bbox) {
  const amenityTags = {
    schools: { key: 'amenity', value: 'school' },
    churches: { key: 'amenity', value: 'place_of_worship' },
    shelters: { key: 'amenity', value: 'shelter' },
    hospitals: { key: 'amenity', value: 'hospital' },
    kindergartens: { key: 'amenity', value: 'kindergarten' },
    bus_stops: { key: 'highway', value: 'bus_stop' },
    supermarkets: { key: 'shop', value: 'supermarket' },
    // museums: { key: 'amenity', value: 'museum' }
  }
  
  const tag = amenityTags[type]
  if (!tag) return []

  // Overpass QL запит — ключ і значення підставляються тут
  const query = `
    [out:json][timeout:25];
    (
      node["${tag.key}"="${tag.value}"](${bbox.south},${bbox.west},${bbox.north},${bbox.east});
      way["${tag.key}"="${tag.value}"](${bbox.south},${bbox.west},${bbox.north},${bbox.east});
      relation["${tag.key}"="${tag.value}"](${bbox.south},${bbox.west},${bbox.north},${bbox.east});
    );
    out center;
  `
  
  try {
    const res = await fetch('https://overpass-api.de/api/interpreter', {
      method: 'POST',
      body: query,
      headers: {
        'Content-Type': 'text/plain'
      }
    })
    if (!res.ok) return []
    const data = await res.json()

    return data.elements.map(el => {
      let lat, lng
      if (el.type === 'node') {
        lat = el.lat
        lng = el.lon
      } else if (el.center) {
        lat = el.center.lat
        lng = el.center.lon
      }
      return {
        id: el.id,
        name: el.tags?.name || `${tag.value}`, // Якщо немає назви, показуємо тип
        lat,
        lng
      }
    })
  } catch (e) {
    console.error(`Помилка завантаження POI: ${type}`, e)
    return []
  }
}


async function renderExtraLayers() {
  if (!map.value) return

  extraMarkers.value.forEach(m => map.value.removeLayer(m))
  extraMarkers.value = []

  const bbox = getBBox()
  if (!bbox) return

  const iconSize = [30, 30]  

  for (const [key, active] of Object.entries(props.layers)) {
    if (active) {
      const pois = await fetchPOI(key, bbox)

      // Іконка для цього типу
      const icon = L.icon({
        iconUrl: `/icons/${key}.png`,  
        iconSize: iconSize,
        iconAnchor: [15, 30], 
        popupAnchor: [0, -30]  
      })

      pois.forEach(poi => {
        const marker = L.marker([poi.lat, poi.lng], { icon })
          .addTo(map.value)
          .bindPopup(poi.name)
        extraMarkers.value.push(marker)
      })
    }
  }
}

function renderMarkers(L) {
  if (!map.value) return

  props.ads.forEach(ad => {
    if (ad.lat && ad.lng) {
      L.marker([ad.lat, ad.lng])
        .addTo(map.value)
        .bindPopup(`<strong>${ad.title}</strong><br>Ціна: ${ad.price} грн`)
    }
  })
}

onMounted(async () => {
  if (!process.client) return
  const L = await import('leaflet')

  map.value = L.map('map').setView([50.45, 30.52], 6)

  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap'
  }).addTo(map.value)

  renderMarkers(L)
  renderExtraLayers()

  map.value.on('moveend', () => {
    renderExtraLayers()
  })

  if (props.interactive) {
    map.value.on('click', (e) => {
      const coords = {
        lat: e.latlng.lat,
        lng: e.latlng.lng
      }

      if (selectedMarker.value) {
        selectedMarker.value.setLatLng([coords.lat, coords.lng])
      } else {
        selectedMarker.value = L.marker([coords.lat, coords.lng], { draggable: false })
          .addTo(map.value)
          .bindPopup('Ваш вибір')
          .openPopup()
      }

      emit('mapClick', coords)
    })
  }
})

watch(() => props.ads, async () => {
  if (!process.client || !map.value) return
  const L = await import('leaflet')
  renderMarkers(L)
}, { immediate: true })

watch(() => props.layers, async () => {
  if (!process.client || !map.value) return
  renderExtraLayers()
}, { deep: true })
</script>

<template>
  <div id="map" class="map-container"></div>
</template>

<style scoped>
.map-container {
  width: 100%;
  height: 400px;
  margin-top: 20px;
  border-radius: 8px;
  overflow: hidden;
}
</style>

<style>
@import "leaflet/dist/leaflet.css";
</style>
