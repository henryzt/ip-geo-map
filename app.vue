<template>
  <div class="flex">
    <Form class="w-72" @on-submit="onSubmit" />
    <Map class="flex-grow" :markerData="longLatIpArr" />
  </div>
</template>

<script setup lang="ts">
const longLatIpArr = ref<{ lng: number; lat: number; ip: string }[]>([]);

const getLongLat = async (ip: string) => {
  const res = await fetch(`https://ipapi.co/${ip}/latlong/`);
  const [lat, lng] = await res.text().then((res) => res.split(","));
  console.log(lng, lat, ip)
  return { lng: Number(lng), lat: Number(lat), ip };
};

const onSubmit = async (ipArr: string[]) => {
  console.log(ipArr);
  longLatIpArr.value = await Promise.all(ipArr.map(getLongLat));
};
</script>
