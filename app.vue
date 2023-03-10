<template>
  <div class="flex">
    <div class="m-5 max-w-xs">
      <div class="my-3 text-3xl">IP Geo Map</div>
      <div>Mapping IP Addresses</div>
      <div class="flex mt-3">
        <input class="input input-bordered w-full mr-2" type="text" placeholder="Google Map API key"
          v-model="apiKeyModel" />
        <button class="btn" @click="setApiKey">Set</button>
      </div>
      <Form class="w-full" @on-submit="onSubmit" />
      <div class="pt-5">Fetched {{ longLatIpArr.length }} out of {{ ips.length }}, rendered
        {{ throttledLongLatIpArr.length }}</div>
      <progress class="progress progress-primary w-full" :value="longLatIpArr.length" :max="ips.length"></progress>
      <div class="opacity-30 text-sm">Results will be cached in local storage for reuse, nothing are stored remotely
      </div>
    </div>
    <ClientOnly>
      <Map class="flex-grow" ref="mapRef" :markerData="throttledLongLatIpArr" />
    </ClientOnly>
  </div>
</template>

<script setup lang="ts">
import { refThrottled, useStorage } from '@vueuse/core'

const mapRef = ref();
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
    return { lng: Number(lng), lat: Number(lat), ip, added: false };
  }
  try {
    const { data } = await useFetch<string>(`https://ipapi.co/${ip}/latlong/`);
    if (!data.value) throw new Error("No data");

    const [lat, lng] = data.value.split(",");
    ipLatLngMap.value = { ...ipLatLngMap.value, [ip]: { lat: Number(lat), lng: Number(lng) } };
    console.log(ip, lat, lng)
    return { lng: Number(lng), lat: Number(lat), ip, added: false };
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
  mapRef.value.cleanMarkers();

  for (const ip of ipArr) {
    const longLat = await getLongLat(ip);
    longLatIpArr.value = [...longLatIpArr.value, longLat];
  }
  throttledLongLatIpArr.value = longLatIpArr.value;
};
</script>
