<template>
  <div class="w-full h-screen bg-black">
    <div class="w-full h-full" id="map"></div>
  </div>
</template>

<script setup lang="ts">
import { Loader } from '@googlemaps/js-api-loader';

onMounted(() => {
  const loader = new Loader({
    apiKey: "",
    version: "weekly",
    libraries: ["places"]
  });

  const mapOptions = {
    center: {
      lat: 0,
      lng: 0
    },
    zoom: 4
  };

  loader
    .load()
    .then((google) => {
      new google.maps.Map(document.getElementById("map"), mapOptions);
    })
    .catch(e => {
      // do something
    });
})
</script>

<style scoped>

</style>