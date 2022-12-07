<script setup>
import { computed, onMounted, ref } from "vue";

const DIR_UP = "UP";
const DIR_DOWN = "DOWN";
const DIR_LEFT = "LEFT";
const DIR_RIGHT = "RIGHT";

const TICKRATE = 1000;

const CANVAS_WIDTH = 1000;
const CANVAS_HEIGHT = 500;
const CANVAS_WIDTH_PX = computed(() => `${CANVAS_WIDTH}px`);
const CANVAS_HEIGHT_PX = computed(() => `${CANVAS_HEIGHT}px`);
// the below "PIXEL" is NxN actual atomic pixels
const PIXEL_SIZE = 5;
// the below "SQUARE" is MxM of the above PIXELs
const SQUARE_SIZE = 2;

let context;
const canvas = ref(null);

// prettier-ignore
const spawnPos = { 
  x: CANVAS_WIDTH / PIXEL_SIZE / SQUARE_SIZE, 
  y: CANVAS_HEIGHT / PIXEL_SIZE / SQUARE_SIZE 
};

// prettier-ignore
const snakeState = {
  coords: [spawnPos, 
  { ...spawnPos, x: spawnPos.x + SQUARE_SIZE }, 
  { ...spawnPos, x: spawnPos.x + 2 * SQUARE_SIZE }, 
  { ...spawnPos, x: spawnPos.x + 3 * SQUARE_SIZE }],
  dir: DIR_LEFT,
};

onMounted(() => {
  context = canvas.value.getContext("2d", {
    alpha: false,
  });

  fillBg("#4F822B");

  drawSnake(11, 11);

  drawGrid({
    drawFirstLine: true,
    color: "#4F822B88",
  });
});

const drawSnake = (x, y) => {
  // TODO draw snake: 1 'pixelset' for tail, 1 for body, one for head. 1 for tongue?

  //HEAD
  // MOUTH
  drawGreenPx(x + 1, y + 1);
  drawGreenPx(x + 1, y + 2);
  drawGreenPx(x + 2, y + 1);
  drawGreenPx(x + 2, y + 2);
  // EYE
  drawGreenPx(x + 3, y + 2);
  drawGreenPx(x + 3, y + 0);
  drawGreenPx(x + 4, y + 1);
  drawGreenPx(x + 4, y + 2);

  //BODY
  // ONE SEGMENT (STRIPED)
  drawGreenPx(x + 5, y + 2);
  drawGreenPx(x + 6, y + 1);
  // ONE SEGMENT (SOLID)
  drawGreenPx(x + 7, y + 1);
  drawGreenPx(x + 7, y + 2);
  drawGreenPx(x + 8, y + 1);
  drawGreenPx(x + 8, y + 2);

  // TAIL
  drawGreenPx(x + 9, y + 2);
  drawGreenPx(x + 10, y + 1);
  drawGreenPx(x + 11, y + 1);
  drawGreenPx(x + 11, y + 2);
  drawGreenPx(x + 12, y + 1);
  drawGreenPx(x + 12, y + 2);
  drawGreenPx(x + 13, y + 2);
  drawGreenPx(x + 14, y + 2);
};

const drawGreenPx = (x, y) => {
  context.fillStyle = "#213404";
  context.fillRect(x * PIXEL_SIZE, y * PIXEL_SIZE, PIXEL_SIZE, PIXEL_SIZE);
};

const fillBg = (color) => {
  context.fillStyle = color;
  context.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
};

const drawGrid = (options) => {
  const { color, drawFirstLine } = options;
  for (let x = drawFirstLine ? 0 : 1; x <= CANVAS_WIDTH; x += PIXEL_SIZE) {
    context.moveTo(x, 0);
    context.lineTo(x, CANVAS_HEIGHT);
  }
  for (let x = drawFirstLine ? 0 : 1; x <= CANVAS_HEIGHT; x += PIXEL_SIZE) {
    context.moveTo(0, x);
    context.lineTo(CANVAS_WIDTH, x);
  }
  context.strokeStyle = color;
  context.stroke();
};
</script>

<template>
  <canvas ref="canvas" :height="CANVAS_HEIGHT" :width="CANVAS_WIDTH" class="mi-auto border border-white" />
</template>

<style>
canvas {
  height: v-bind(CANVAS_HEIGHT_PX);
  width: v-bind(CANVAS_WIDTH_PX);
}
</style>
