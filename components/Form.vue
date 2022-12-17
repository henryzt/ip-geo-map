<template>
  <div class="flex flex-col">
    <textarea class="textarea textarea-bordered my-2 h-56" v-model="ips"
      placeholder="IP addresses, separated by newline" />
    <button class="btn" @click="convertIp">Fetch</button>
  </div>
</template>

<script setup lang="ts">
import { useStorage } from '@vueuse/core'

const emits = defineEmits(["onSubmit"]);
const ips = useStorage("ipInput", "");

const convertIp = () => {
  const ipArr = ips.value.split("\n").map((ip) => ip.trim());
  emits("onSubmit", ipArr);
};
</script>

<style scoped>

</style>