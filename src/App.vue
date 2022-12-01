<script setup>
import { computed, onMounted, ref } from "vue";

const CANVAS_WIDTH = 1000;
const CANVAS_HEIGHT = 500;
const CANVAS_WIDTH_PX = computed(() => `${CANVAS_WIDTH}px`);
const CANVAS_HEIGHT_PX = computed(() => `${CANVAS_HEIGHT}px`);
const CELL_SIZE = 5;

const canvas = ref(null);

onMounted(() => {
  const gameLayerCtx = canvas.value.getContext("2d", {
    alpha: false,
  });

  fillBg(gameLayerCtx, "#4F822B");

  drawGrid(gameLayerCtx, {
    drawFirstLine: true,
    color: "#00000088",
  });
});

const fillBg = (context, color) => {
  context.fillStyle = color;
  context.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
};

const drawGrid = (context, options) => {
  const { color, drawFirstLine } = options;
  for (
    let x = drawFirstLine ? 0 : 1;
    x <= CANVAS_WIDTH;
    x += CELL_SIZE
  ) {
    context.moveTo(x, 0);
    context.lineTo(x, CANVAS_HEIGHT);
  }
  for (
    let x = drawFirstLine ? 0 : 1;
    x <= CANVAS_HEIGHT;
    x += CELL_SIZE
  ) {
    context.moveTo(0, x);
    context.lineTo(CANVAS_WIDTH, x);
  }
  context.strokeStyle = color;
  context.stroke();
};
</script>

<template>
  <canvas
    ref="canvas"
    :height="CANVAS_HEIGHT"
    :width="CANVAS_WIDTH"
    class="mi-auto border border-white"
  />
</template>

<style>
canvas {
  height: v-bind(CANVAS_HEIGHT_PX);
  width: v-bind(CANVAS_WIDTH_PX);
}
</style>
