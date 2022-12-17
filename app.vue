<template>
  <div class="flex">
    <div class="m-5">
      <div class="my-3 text-3xl">IP Geo Map</div>
      <div class="flex">
        <input class="input input-bordered w-full mr-2" type="text" placeholder="Google Map API key"
          v-model="apiKeyModel" />
        <button class="btn" @click="setApiKey">Set</button>
      </div>
      <Form class="w-72" @on-submit="onSubmit" />
      <div class="py-5">Fetched {{ longLatIpArr.length }} out of {{ ips.length }}, rendered
        {{ throttledLongLatIpArr.length }}</div>
    </div>
    <ClientOnly>
      <Map class="flex-grow" :markerData="throttledLongLatIpArr" />
    </ClientOnly>
  </div>
</template>

<script setup lang="ts">
import { refThrottled, useStorage } from '@vueuse/core'

const ips = ref<string[]>([]);
const longLatIpArr = ref<{ lng: number; lat: number; ip: string }[]>([]);
const throttledLongLatIpArr = refThrottled(longLatIpArr, 3000);
const ipLatLngMap = useStorage<{ [key: string]: { lng: number; lat: number } }>("ips", {})
const apiKeyModel = useStorage("gMapApiKey", "");

const setApiKey = () => {
  window.location.reload();
}

const getLongLat = async (ip: string) => {
  const fromStorage = ipLatLngMap.value[ip]
  if (fromStorage) {
    const { lat, lng } = fromStorage;
    return { lng: Number(lng), lat: Number(lat), ip };
  }
  try {
    const res = await fetch(`https://ipapi.co/${ip}/latlong/`);
    const [lat, lng] = await res.text().then((res) => res.split(","));
    console.log(ip, lat, lng)
    ipLatLngMap.value = { ...ipLatLngMap.value, [ip]: { lat: Number(lat), lng: Number(lng) } };
    return { lng: Number(lng), lat: Number(lat), ip };
  } catch (e) {
    console.error(e);
    return { lng: 0, lat: 0, ip };
  }
};

const onSubmit = async (ipArr: string[]) => {
  console.log(ipArr);
  ips.value = ipArr;
  longLatIpArr.value = [];
  throttledLongLatIpArr.value = [];

  for (const ip of ipArr) {
    const longLat = await getLongLat(ip);
    longLatIpArr.value = [...longLatIpArr.value, longLat];
  }
  throttledLongLatIpArr.value = longLatIpArr.value;
};
</script>
