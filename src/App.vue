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

  drawSnake(gameLayerCtx, 11, 11, "#213404");

  drawGrid(gameLayerCtx, {
    drawFirstLine: true,
    color: "#4F822B88",
  });
});

const drawSnake = (context, x, y, color) => {
  // TODO draw snake: 1 'pixelset' for tail, 1 for body, one for head. 1 for tongue?

  //HEAD
  // MOUTH
  drawPixel(context, x + 1, y + 1, color);
  drawPixel(context, x + 1, y + 2, color);
  drawPixel(context, x + 2, y + 1, color);
  drawPixel(context, x + 2, y + 2, color);
  // EYE
  drawPixel(context, x + 3, y + 2, color);
  drawPixel(context, x + 3, y + 0, color);
  drawPixel(context, x + 4, y + 1, color);
  drawPixel(context, x + 4, y + 2, color);

  //BODY
  // ONE SEGMENT (STRIPED)
  drawPixel(context, x + 5, y + 2, color);
  drawPixel(context, x + 6, y + 1, color);
  // ONE SEGMENT (SOLID)
  drawPixel(context, x + 7, y + 1, color);
  drawPixel(context, x + 7, y + 2, color);
  drawPixel(context, x + 8, y + 1, color);
  drawPixel(context, x + 8, y + 2, color);

  // TAIL
  drawPixel(context, x + 9, y + 2, color);
  drawPixel(context, x + 10, y + 1, color);
  drawPixel(context, x + 11, y + 1, color);
  drawPixel(context, x + 11, y + 2, color);
  drawPixel(context, x + 12, y + 1, color);
  drawPixel(context, x + 12, y + 2, color);
  drawPixel(context, x + 13, y + 2, color);
  drawPixel(context, x + 14, y + 2, color);
};

const drawPixel = (context, x, y, color) => {
  context.fillStyle = color;
  context.fillRect(
    x * CELL_SIZE,
    y * CELL_SIZE,
    CELL_SIZE,
    CELL_SIZE
  );
};

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
