<script setup>
import { computed, onMounted, ref } from "vue";

const width = 1000;
const height = 500;
const widthpx = computed(() => `${width}px`);
const heightpx = computed(() => `${height}px`);

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
  // Gap
  const cellSize = 5;
  // Draw first line
  const drawFirst = true;
  for (let x = drawFirst ? 0 : 1; x <= bw; x += cellSize) {
    context.moveTo(x, 0);
    context.lineTo(x, bh);
  }
  for (let x = drawFirst ? 0 : 1; x <= bh; x += cellSize) {
    context.moveTo(0, x);
    context.lineTo(bw, x);
  }
  context.strokeStyle = "black";
  context.stroke();
});
</script>

<template>
  <canvas
    ref="canvas"
    :height="height"
    :width="width"
    class="mi-auto border border-black"
  />
</template>

<style>
canvas {
  height: v-bind(heightpx);
  width: v-bind(widthpx);
}
</style>
