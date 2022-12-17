<template>
  <div>Enter a few ip addresses</div>
  <textarea :value="ips" @input="onInput" />
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