<script setup>
defineProps({
  msg: String,
});
</script>

<template>
  <h1
    class="bg-gradient-to-r from-green-100 to-green-600 bg-clip-text text-8xl font-black text-transparent selection:bg-transparent"
  >
    <slot></slot>
  </h1>
</template>

<style scoped></style>
