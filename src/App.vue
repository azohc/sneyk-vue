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

  drawSnake();

  setInterval(() => {
    // DETERMINE NEW HEAD POSITION
    const currentHead = snakeState.coords[0];
    if (snakeState.dir === DIR_LEFT) {
      // prettier-ignore
      snakeState.coords.unshift({
        x: currentHead.x > 0
          ? currentHead.x - SQUARE_SIZE
          : CANVAS_WIDTH / PIXEL_SIZE - SQUARE_SIZE,
        y: currentHead.y,
      });
    }
    snakeState.coords.pop();

    // TODO DONT RESET, JUST ERASE SNAKE USING COORDS ARRAY
    reset();

    drawSnake();
  }, TICKRATE);
  // drawGrid({
  //   drawFirstLine: true,
  //   color: "#4F822B88",
  // });
});

const drawSquare = (x, y, stripe) => {
  if (stripe) {
    /// ONE SQUARE SLANT = \
    // bot left
    drawGreenPx(wrapxc(x), wrapyc(y + 2));
    // top right
    drawGreenPx(wrapxc(x + 1), wrapyc(y + 1));
  } else {
    drawGreenPx(x, y + 1);
    drawGreenPx(x, y + 2);
    drawGreenPx(x + 1, y + 1);
    drawGreenPx(x + 1, y + 2);
  }
};

const drawSnake = () => {
  const X_CELLS = CANVAS_WIDTH / PIXEL_SIZE;
  const Y_CELLS = CANVAS_HEIGHT / PIXEL_SIZE;
  const wrapxc = (xc) => (xc >= X_CELLS ? xc - X_CELLS : xc);
  const wrapyc = (yc) => (yc >= Y_CELLS ? yc - X_CELLS : yc);

  const { coords, dir } = snakeState;
  const len = coords.length;
  const head = coords[0];
  const tail = coords[len - 1];

  /// HEAD
  // MOUTH
  drawSquare(head.x, head.y);
  // EYE
  drawGreenPx(head.x + SQUARE_SIZE, head.y + 2);
  drawGreenPx(head.x + SQUARE_SIZE, head.y + 0);
  drawGreenPx(head.x + SQUARE_SIZE + 1, head.y + 1);
  drawGreenPx(head.x + SQUARE_SIZE + 1, head.y + 2);

  // //BODY
  // // ONE SEGMENT (STRIPED)
  // drawGreenPx(x + 5, y + 2);
  // drawGreenPx(x + 6, y + 1);
  // // ONE SEGMENT (SOLID)
  // drawGreenPx(x + 7, y + 1);
  // drawGreenPx(x + 7, y + 2);
  // drawGreenPx(x + 8, y + 1);
  // drawGreenPx(x + 8, y + 2);

  // // TAIL
  // drawGreenPx(x + 9, y + 2);
  // drawGreenPx(x + 10, y + 1);
  // drawGreenPx(x + 11, y + 1);
  // drawGreenPx(x + 11, y + 2);
  // drawGreenPx(x + 12, y + 1);
  // drawGreenPx(x + 12, y + 2);
  // drawGreenPx(x + 13, y + 2);
  // drawGreenPx(x + 14, y + 2);
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

const reset = () => {
  context.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
  fillBg("#4F822B");
};
</script>

<template>
  <button @click="reset">reset</button>
  <canvas ref="canvas" :height="CANVAS_HEIGHT" :width="CANVAS_WIDTH" class="mi-auto border border-white" />
</template>

<style>
canvas {
  height: v-bind(CANVAS_HEIGHT_PX);
  width: v-bind(CANVAS_WIDTH_PX);
}
</style>
