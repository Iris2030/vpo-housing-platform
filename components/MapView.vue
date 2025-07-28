<script setup>
import { onMounted, watch, ref } from 'vue'
import { useRouter } from 'vue-router'
import L from 'leaflet'
import 'leaflet/dist/leaflet.css'

const router = useRouter()

const props = defineProps({
  ads: Array,
  layers: Object,
  filters: Object,
  interactive: Boolean
})

const emit = defineEmits(['mapClick'])

const map = ref(null)
const selectedMarker = ref(null)
const extraMarkers = ref([])
const adMarkers = ref([])

delete L.Icon.Default.prototype._getIconUrl;

L.Icon.Default.mergeOptions({
  iconRetinaUrl: new URL('leaflet/dist/images/marker-icon-2x.png', import.meta.url).href,
  iconUrl: new URL('leaflet/dist/images/marker-icon.png', import.meta.url).href,
  shadowUrl: new URL('leaflet/dist/images/marker-shadow.png', import.meta.url).href,
})

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
  }
  
  const tag = amenityTags[type]
  if (!tag) return []
  
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
    const res = await fetch('https://overpass.kumi.systems/api/interpreter', {
      method: 'POST',
      body: query,
      headers: { 'Content-Type': 'text/plain' }
    })
    
    if (!res.ok) {
      console.warn(`[${type}] Overpass API error: ${res.status}`)
      return []
    }
    
    const data = await res.json()
    return data.elements.map(el => {
      const lat = el.type === 'node' ? el.lat : el.center?.lat
      const lng = el.type === 'node' ? el.lon : el.center?.lon
      return { id: el.id, name: el.tags?.name || type, lat, lng }
    }).filter(p => p.lat && p.lng)
  } catch (e) {
    console.error(`POI fetch error for ${type}:`, e)
    return []
  }
}

async function renderExtraLayers() {
  if (!map.value || !map.value._loaded) {
    console.log('Map not loaded yet, skipping renderExtraLayers')
    return
  }
  
  try { map.value.closePopup() } catch (_) {}
  
  extraMarkers.value.forEach(marker => {
    try {
      if (map.value.hasLayer(marker)) {
        map.value.removeLayer(marker)
      }
      marker.off()
    } catch (e) {
      console.error('Error removing marker:', e)
    }
  })
  extraMarkers.value = []
  
  const bbox = getBBox()
  if (!bbox) return
  
  for (const [key, active] of Object.entries(props.layers || {})) {
    if (!active) continue
    
    console.log(`Fetching POI for layer ${key}`)
    const pois = await fetchPOI(key, bbox)
    
    const icon = L.icon({
      iconUrl: `/icons/${key}.png`,
      iconSize: [30, 30],
      iconAnchor: [15, 30],
      popupAnchor: [0, -30],
    })
    
    pois.forEach(poi => {
      const marker = L.marker([poi.lat, poi.lng], { icon }).bindPopup(poi.name, { autoClose: true, closeOnClick: true })
      marker.addTo(map.value)
      extraMarkers.value.push(marker)
      // <-- НЕ відкривати marker.openPopup()
    })
  }
}


function renderMarkers(L) {
  if (!map.value) return;
  
  // ⛔️ ЦЕ ОБОВ'ЯЗКОВО
  try { map.value.closePopup() } catch (_) {}
  
  adMarkers.value.forEach(marker => {
    try {
      marker.closePopup?.()
      marker.off()
      map.value.removeLayer(marker)
    } catch (_) {}
  })
  adMarkers.value = []
  
  const filtered = props.ads.filter(ad => {
    if (!ad.lat || !ad.lng) return false
    if (props.filters?.onlyFree && ad.price > 0) return false
    if (props.filters?.rooms === 4) return ad.rooms >= 4
    if (props.filters?.rooms) return ad.rooms === props.filters.rooms
    return true
  })
  
  filtered.forEach(ad => {
    const marker = L.marker([ad.lat, ad.lng])
    .addTo(map.value)
    .bindPopup(`
      <div class="popup-link" data-id="${ad.id}" style="cursor:pointer;">
        <strong>${ad.title}</strong><br>
        Ціна: ${ad.price === 0 ? 'Безкоштовно' : ad.price + ' грн'}<br>
        <span style="color:#1e90ff;text-decoration:underline;">Переглянути</span>
      </div>
`)
    adMarkers.value.push(marker)
  })
}

onMounted(async () => {
  if (!process.client) return
  const L = await import('leaflet')
  
  document.addEventListener('click', (e) => {
    const target = e.target.closest('.popup-link')
    if (target?.dataset?.id) {
      e.preventDefault()
      router.push(`/ad/${target.dataset.id}`)
    }
  })

  map.value = L.map('map', { zoomAnimation: false })   
    .setView([50.45, 30.52], 6)
  
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
      const coords = { lat: e.latlng.lat, lng: e.latlng.lng }
      
      if (selectedMarker.value) {
        selectedMarker.value.setLatLng([coords.lat, coords.lng])
      } else {
        selectedMarker.value = L.marker([coords.lat, coords.lng])
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

watch(() => props.filters, async () => {
  if (!process.client || !map.value) return
  const L = await import('leaflet')
  renderMarkers(L)
}, { deep: true })

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
