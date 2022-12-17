<template>
  <div class="m-5 flex flex-col">
    <div class="my-2 text-3xl">IP Geo Map</div>
    <textarea class="textarea textarea-bordered my-2" :value="ips" @input="onInput"
      placeholder="IP addresses, separated by comma" />
    <button class="btn">OK</button>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  ips: string;
}>();
const emits = defineEmits(["update:ips"]);

const onInput = (e: any) => {
  emits("update:ips", e?.target?.value);
};
</script>

<style scoped>

</style>