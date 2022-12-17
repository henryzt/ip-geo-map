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

interface ILongLatIp {
  lng: number;
  lat: number;
  ip: string;
}

const ips = ref<string[]>([]);
const longLatIpArr = ref<ILongLatIp[]>([]);
const throttledLongLatIpArr = refThrottled(longLatIpArr, 3000);
const ipLatLngMap = useStorage<{ [key: string]: { lng: number; lat: number } }>("ips", {})
const apiKeyModel = useStorage("gMapApiKey", "");

const setApiKey = () => {
  window.location.reload();
}

const getLongLat = async (ip: string): Promise<ILongLatIp> => {
  const fromStorage = ipLatLngMap.value[ip]
  if (fromStorage) {
    const { lat, lng } = fromStorage;
    return { lng: Number(lng), lat: Number(lat), ip };
  }
  try {
    const { data } = await useFetch<string>(`https://ipapi.co/${ip}/latlong/`);
    if (!data.value) throw new Error("No data");

    const [lat, lng] = data.value.split(",");
    ipLatLngMap.value = { ...ipLatLngMap.value, [ip]: { lat: Number(lat), lng: Number(lng) } };
    console.log(ip, lat, lng)
    return { lng: Number(lng), lat: Number(lat), ip };
  } catch (e) {
    console.error(e);
    // retry after 1.5s
    console.warn("retrying", ip)
    await new Promise((resolve) => setTimeout(resolve, 1500));
    return getLongLat(ip);
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
