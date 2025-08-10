<script setup>
import { onMounted, watch, ref, toRaw } from 'vue'
import { useRouter } from 'vue-router'
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

let adLayer = null
let extraLayer = null
let Leaflet = null

function debounce(fn, wait = 200) {
  let t
  return (...args) => {
    clearTimeout(t)
    t = setTimeout(() => fn(...args), wait)
  }
}

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
  if (!map.value || !map.value._loaded) return

  const MIN_ZOOM_FOR_POI = 13
  if (map.value.getZoom() < MIN_ZOOM_FOR_POI) {
    extraLayer?.clearLayers()
    return
  }

  if (map.value._animatingZoom) {
    map.value.once('zoomend', () => renderExtraLayers())
    return
  }

  try { map.value.closePopup() } catch (_) {}

  const bbox = getBBox()
  if (!bbox) return

  if (!extraLayer) extraLayer = Leaflet.layerGroup().addTo(toRaw(map.value))
  extraLayer.clearLayers()

  for (const [key, active] of Object.entries(props.layers || {})) {
    if (!active) continue

    const pois = await fetchPOI(key, bbox)
    const icon = Leaflet.icon({
      iconUrl: `/icons/${key}.png`,
      iconSize: [30, 30],
      iconAnchor: [15, 30],
      popupAnchor: [0, -30],
    })

    pois.forEach(poi => {
      const marker = Leaflet.marker([poi.lat, poi.lng], { icon })
        .bindPopup(poi.name, { autoClose: true, closeOnClick: true })
      extraLayer.addLayer(marker)
    })
  }
}

function renderMarkers() {
  if (!map.value) return

  try { map.value.closePopup() } catch (_) {}

  if (!adLayer) adLayer = Leaflet.layerGroup().addTo(toRaw(map.value))
  adLayer.clearLayers()

  const filtered = (props.ads || []).filter(ad => {
    if (!ad.lat || !ad.lng) return false
    if (props.filters?.onlyFree && ad.price > 0) return false
    if (props.filters?.rooms === 4) return ad.rooms >= 4
    if (props.filters?.rooms) return ad.rooms === props.filters.rooms
    return true
  })

  filtered.forEach(ad => {
    const marker = Leaflet.marker([ad.lat, ad.lng])
      .bindPopup(`
        <div class="popup-link" data-id="${ad.id}" style="cursor:pointer;">
          <strong>${ad.title}</strong><br>
          Ціна: ${ad.price === 0 ? 'Безкоштовно' : ad.price + ' грн'}<br>
          <span style="color:#1e90ff;text-decoration:underline;">Переглянути</span>
        </div>
      `)
    adLayer.addLayer(marker)
  })
}

onMounted(async () => {
  if (!process.client) return

  const mod = await import('leaflet')
  Leaflet = mod.default || mod

  if (Leaflet.Popup && Leaflet.Popup.prototype) {
  const origPopupAnimate = Leaflet.Popup.prototype._animateZoom;
  Leaflet.Popup.prototype._animateZoom = function (e) {
    if (!this._map) return;         
    return origPopupAnimate && origPopupAnimate.call(this, e);
  };
}

if (Leaflet.Tooltip && Leaflet.Tooltip.prototype) {
  const origTooltipAnimate = Leaflet.Tooltip.prototype._animateZoom;
  Leaflet.Tooltip.prototype._animateZoom = function (e) {
    if (!this._map) return;            
    return origTooltipAnimate && origTooltipAnimate.call(this, e);
  };
}

  delete Leaflet.Icon.Default.prototype._getIconUrl;
  Leaflet.Icon.Default.mergeOptions({
    iconRetinaUrl: new URL('leaflet/dist/images/marker-icon-2x.png', import.meta.url).href,
    iconUrl: new URL('leaflet/dist/images/marker-icon.png', import.meta.url).href,
    shadowUrl: new URL('leaflet/dist/images/marker-shadow.png', import.meta.url).href,
  })

  map.value = Leaflet.map('map', { zoomAnimation: true, markerZoomAnimation: false })
    .setView([50.45, 30.52], 6)

  Leaflet.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap'
  }).addTo(map.value)

  adLayer = Leaflet.layerGroup().addTo(toRaw(map.value))
  extraLayer = Leaflet.layerGroup().addTo(toRaw(map.value))

  renderMarkers()
  renderExtraLayers()

  const scheduleRenderExtraLayers = debounce(() => {
    if (!map.value) return
    if (map.value._animatingZoom) {
      map.value.once('zoomend', () => renderExtraLayers())
      return
    }
    renderExtraLayers()
  }, 250)

  map.value.on('moveend', scheduleRenderExtraLayers)
  map.value.on('zoomend', scheduleRenderExtraLayers)

  document.addEventListener('click', (e) => {
    const target = e.target.closest('.popup-link')
    if (target?.dataset?.id) {
      e.preventDefault()
      router.push(`/ad/${target.dataset.id}`)
    }
  })

  if (props.interactive) {
    map.value.on('click', (e) => {
      const coords = { lat: e.latlng.lat, lng: e.latlng.lng }

      if (selectedMarker.value) {
        selectedMarker.value.setLatLng([coords.lat, coords.lng])
      } else {
        selectedMarker.value = Leaflet.marker([coords.lat, coords.lng])
          .addTo(toRaw(map.value))
          .bindPopup('Ваш вибір')
          .openPopup()
      }

      emit('mapClick', coords)
    })
  }
})

watch(() => props.ads, async () => {
  if (!process.client || !map.value) return
  renderMarkers()
}, { immediate: true })

watch(() => props.filters, async () => {
  if (!process.client || !map.value) return
  renderMarkers()
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
