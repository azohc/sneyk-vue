<script setup>
import { onMounted, ref } from "vue";
import Title from "./components/Title.vue";

const canvas = ref(null);
onMounted(() => {
  console.log("mounted", canvas.value);
  const context = canvas.value.getContext("2d");
  console.log("mounted", context);
  const canvasBounds = canvas.value.getBoundingClientRect();
  // Box width
  const bw = canvasBounds.width;
  // Box height
  const bh = canvasBounds.height;
  // Padding
  const pad = 0;
  // Gap
  const cellSize = 4;
  // Draw first line
  const drawFirst = false;
  for (let x = drawFirst ? 0 : 1; x <= bw; x += cellSize) {
    context.moveTo(0.5 + x + pad, pad);
    context.lineTo(0.5 + x + pad, bh + pad);
  }
  for (let x = drawFirst ? 0 : 1; x <= bh; x += cellSize) {
    context.moveTo(pad, 0.5 + x + pad);
    context.lineTo(bw + pad, 0.5 + x + pad);
  }
  context.strokeStyle = "black";
  context.stroke();
});
</script>

<template>
  <div class="text-center selection:bg-green-100">
    <Title> sneyk </Title>
  </div>
  <canvas ref="canvas" class="h-3/5 border border-green-900" />
</template>

<style></style>
