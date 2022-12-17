<template>
  <div class="w-full h-screen bg-black">
    <div class="w-full h-full" id="map"></div>
  </div>
</template>

<script setup lang="ts">
import { Loader } from '@googlemaps/js-api-loader';
import { MarkerClusterer } from "@googlemaps/markerclusterer";

const props = defineProps<{
  markerData: ILongLatIp[];
}>();

const addedIpMap = ref<{ [key: string]: boolean }>({});

let mapObj: google.maps.Map;
let infoWindow: google.maps.InfoWindow;
let markerClusterer: MarkerClusterer;

const mapOptions = {
  center: { lat: 42.0902, lng: -101.712 },
  zoom: 4
};

onMounted(() => {
  const loader = new Loader({
    apiKey: localStorage.getItem("gMapApiKey") || "",
    version: "weekly",
    libraries: ["places"]
  });

  loader
    .load()
    .then((google) => {
      // @ts-expect-error
      mapObj = new google.maps.Map(document.getElementById("map"), mapOptions);

      infoWindow = new google.maps.InfoWindow({
        content: "",
        disableAutoPan: true,
      });

      markerClusterer = new MarkerClusterer({ markers: [], map: mapObj });
    })
    .catch(e => {
      console.error(e)
    });
})

watch(() => props.markerData, () => {
  const markerData = props.markerData;
  if (markerData && mapObj) {
    const markers = []

    for (const position of markerData) {
      if (addedIpMap.value[position.ip]) continue;

      const label = position.ip;
      const marker = new google.maps.Marker({
        position: { lat: position.lat, lng: position.lng },
      });

      marker.addListener("click", () => {
        infoWindow.setContent(label);
        infoWindow.open(mapObj, marker);
      });

      addedIpMap.value[position.ip] = true;

      markers.push(marker)
    }

    markerClusterer.addMarkers(markers);

  }
});
</script>

<style scoped>

</style>