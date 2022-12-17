<template>
  <div class="w-full h-screen bg-black">
    <div class="w-full h-full" id="map"></div>
  </div>
</template>

<script setup lang="ts">
import { Loader } from '@googlemaps/js-api-loader';
import { MarkerClusterer } from "@googlemaps/markerclusterer";

const props = defineProps<{
  markerData: { lng: number; lat: number; ip: string }[];
}>();

const googleMaps = ref<typeof google.maps | undefined>();
const mapObj = ref();

const mapOptions = {
  center: { lat: 42.0902, lng: -101.712 },
  zoom: 4
};


onMounted(() => {
  const loader = new Loader({
    apiKey: "",
    version: "weekly",
    libraries: ["places"]
  });

  loader
    .load()
    .then((google) => {
      googleMaps.value = google.maps;
      mapObj.value = new google.maps.Map(document.getElementById("map"), mapOptions);
    })
    .catch(e => {
      console.error(e)
    });
})

watch(() => props.markerData, () => {
  const markerData = props.markerData;
  console.log('henryDebug markerData', JSON.stringify(markerData));
  if (markerData && mapObj.value) {
    const infoWindow = new google.maps.InfoWindow({
      content: "",
      disableAutoPan: true,
    });

    // Create an array of alphabetical characters used to label the markers.
    const labels = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";

    // Add some markers to the map.
    const markers = markerData.map((position, i) => {
      const label = labels[i % labels.length];
      const marker = new google.maps.Marker({
        position,
        label,
      });

      // markers can only be keyboard focusable when they have click listeners
      // open info window when marker is clicked
      marker.addListener("click", () => {
        infoWindow.setContent(label);
        infoWindow.open(mapObj.value, marker);
      });

      return marker;
    });

    // Add a marker clusterer to manage the markers.
    new MarkerClusterer({ markers, map: mapObj.value });
  }
});
</script>

<style scoped>

</style>