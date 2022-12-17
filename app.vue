<template>
  <div class="flex">
    <div>
      <Form class="w-72" @on-submit="onSubmit" />
      <div class="px-5">Fetched {{ longLatIpArr.length }} out of {{ ips.length }}, rendered
        {{ throttledLongLatIpArr.length }}</div>
    </div>
    <Map class="flex-grow" :markerData="throttledLongLatIpArr" />
  </div>
</template>

<script setup lang="ts">
const ips = ref<string[]>([]);
const longLatIpArr = ref<{ lng: number; lat: number; ip: string }[]>([]);
const throttledLongLatIpArr = ref<{ lng: number; lat: number; ip: string }[]>([]);
const ipLatLngMap = ref<{ [key: string]: { lng: number; lat: number } }>({});

const throttle = (fn: Function, limit: number) => {
  let inThrottle: boolean;
  return function () {
    const args = arguments;
    // @ts-expect-error
    const context = this;
    if (!inThrottle) {
      fn.apply(context, args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
};

const throttledLongLatIpArrSetter = throttle(
  () => {
    console.log("throttle")
    throttledLongLatIpArr.value = longLatIpArr.value;

    localStorage.setItem("ips", JSON.stringify(ipLatLngMap.value));
  },
  3000
);



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
  // longLatIpArr.value = await Promise.all(ipArr.map(getLongLat));
  for (const ip of ipArr) {
    const longLat = await getLongLat(ip);
    longLatIpArr.value = [...longLatIpArr.value, longLat];
    throttledLongLatIpArrSetter();
  }
  throttledLongLatIpArr.value = longLatIpArr.value;
};

onMounted(() => {
  const ipsFromLocalStorage = localStorage.getItem("ips");
  if (ipsFromLocalStorage) {
    ipLatLngMap.value = JSON.parse(ipsFromLocalStorage);
  }
});
</script>
