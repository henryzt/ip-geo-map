<template>
  <div class="flex">
    <Form class="w-72" v-model:ips="ips" />
    <Map class="flex-grow" />
  </div>
</template>

<script setup lang="ts">
const ips = ref("");
</script>
