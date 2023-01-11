<script setup>
import { computed, onMounted, ref } from "vue";

const DIR_UP = "UP";
const DIR_DOWN = "DOWN";
const DIR_LEFT = "LEFT";
const DIR_RIGHT = "RIGHT";

const TICKRATE = 150;

const CANVAS_WIDTH = 1000;
const CANVAS_HEIGHT = 500;
const CANVAS_WIDTH_PX = computed(() => `${CANVAS_WIDTH}px`);
const CANVAS_HEIGHT_PX = computed(() => `${CANVAS_HEIGHT}px`);
// the below "PIXEL" is NxN actual atomic pixels
const PIXEL_SIZE = 5;
// the below "SQUARE" is MxM of the above PIXELs
const SQUARE_SIZE = 2;

let playing;
let context;
const canvas = ref(null);

// prettier-ignore
const spawnPos = {
  x: CANVAS_WIDTH / PIXEL_SIZE / SQUARE_SIZE,
  y: CANVAS_HEIGHT / PIXEL_SIZE / SQUARE_SIZE,
};
const snakeState = ref({
  coords: [
    spawnPos,
    { ...spawnPos, x: spawnPos.x - SQUARE_SIZE },
    { ...spawnPos, x: spawnPos.x - 2 * SQUARE_SIZE },
    { ...spawnPos, x: spawnPos.x - 3 * SQUARE_SIZE },
  ],
  dir: DIR_RIGHT,
});
const latestKeydown = ref(null);

onMounted(() => {
  context = canvas.value.getContext("2d", {
    alpha: false,
  });

  fillBg("#4F822B");

  drawSnake();

  playing = setInterval(onTick, TICKRATE);

  drawGrid({
    drawFirstLine: true,
    color: "#4F822B88",
  });
});

const onTick = () => {
  // consume whatever the latest keydown is
  if (latestKeydown.value) {
    snakeState.value.dir = latestKeydown.value;
    latestKeydown.value = null;
  }
  // DETERMINE NEW HEAD POSITION
  unshiftNewHeadCoord();
  // POP TAIL // TODO UNLESS ATE APPLE ON THIS TICK
  const pop = snakeState.value.coords.pop();

  // TODO DONT RESET, JUST ERASE SNAKE USING COORDS ARRAY
  reset();

  drawSnake();
};

const drawSquare = (x, y, color) => {
  drawPx(x, y, color);
  drawPx(x, y + 1, color);
  drawPx(x + 1, y, color);
  drawPx(x + 1, y + 1, color);
};

const drawSnake = () => {
  const X_CELLS = CANVAS_WIDTH / PIXEL_SIZE;
  const Y_CELLS = CANVAS_HEIGHT / PIXEL_SIZE;
  const wrapxc = (xc) => {
    if (xc >= X_CELLS) return xc - X_CELLS;
    if (xc < 0) return xc + X_CELLS;
    return xc;
  };
  const wrapyc = (yc) => {
    if (yc >= Y_CELLS) return yc - Y_CELLS;
    if (yc < 0) return yc + Y_CELLS;
    return yc;
  };

  const { coords } = snakeState.value;
  const len = coords.length;
  const head = coords[0];

  drawSquare(head.x, head.y);
  coords
    .slice(1, len - 1)
    .forEach((coord) =>
      drawSquare(wrapxc(coord.x), wrapyc(coord.y), "red")
    );
};

function unshiftNewHeadCoord() {
  const head = snakeState.value.coords[0];
  switch (snakeState.value.dir) {
    case DIR_UP:
      snakeState.value.coords.unshift({
        x: head.x,
        y:
          head.y > 0
            ? head.y - SQUARE_SIZE
            : CANVAS_HEIGHT / PIXEL_SIZE - SQUARE_SIZE,
      });
      break;
    case DIR_DOWN:
      snakeState.value.coords.unshift({
        x: head.x,
        y:
          head.y + SQUARE_SIZE < CANVAS_HEIGHT / PIXEL_SIZE
            ? head.y + SQUARE_SIZE
            : 0,
      });
      break;
    case DIR_LEFT:
      snakeState.value.coords.unshift({
        x:
          head.x > 0
            ? head.x - SQUARE_SIZE
            : CANVAS_WIDTH / PIXEL_SIZE - SQUARE_SIZE,
        y: head.y,
      });
      break;
    case DIR_RIGHT:
      snakeState.value.coords.unshift({
        x:
          head.x + SQUARE_SIZE < CANVAS_WIDTH / PIXEL_SIZE
            ? head.x + SQUARE_SIZE
            : 0,
        y: head.y,
      });
      break;
  }
}

const drawPx = (x, y, color) => {
  context.fillStyle = color ? color : "#213404";
  context.fillRect(
    x * PIXEL_SIZE,
    y * PIXEL_SIZE,
    PIXEL_SIZE,
    PIXEL_SIZE
  );
};

const fillBg = (color) => {
  context.fillStyle = color;
  context.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
};

const drawGrid = (options) => {
  const { color, drawFirstLine } = options;
  for (
    let x = drawFirstLine ? 0 : 1;
    x <= CANVAS_WIDTH;
    x += PIXEL_SIZE
  ) {
    context.moveTo(x, 0);
    context.lineTo(x, CANVAS_HEIGHT);
  }
  for (
    let x = drawFirstLine ? 0 : 1;
    x <= CANVAS_HEIGHT;
    x += PIXEL_SIZE
  ) {
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

const togglepause = () => {
  if (playing == null) {
    playing = setInterval(onTick, TICKRATE);
  } else {
    playing = clearInterval(playing);
  }
};
const onkeydown = (event) => {
  if (event.code === "Space") {
    togglepause();
    return;
  }
  const ACCEPTED_DIR_KEYCODES = [
    "ArrowUp",
    "ArrowDown",
    "ArrowLeft",
    "ArrowRight",
    "KeyW",
    "KeyS",
    "KeyA",
    "KeyD",
    "KeyK",
    "KeyJ",
    "KeyH",
    "KeyL",
  ];
  const i = ACCEPTED_DIR_KEYCODES.indexOf(event.code);
  switch (i % 4) {
    case 0:
      latestKeydown.value = DIR_UP;
      break;
    case 1:
      latestKeydown.value = DIR_DOWN;
      break;
    case 2:
      latestKeydown.value = DIR_LEFT;
      break;
    case 3:
      latestKeydown.value = DIR_RIGHT;
      break;
    default:
  }
};
</script>

<template>
  <canvas
    autofocus
    tabindex="0"
    @keydown="onkeydown"
    ref="canvas"
    :height="CANVAS_HEIGHT"
    :width="CANVAS_WIDTH"
    class="mi-auto border-2"
  />
  <button @click="togglepause">
    {{ playing ? "pause" : "resume" }}
  </button>
  <code class="text-center">
    {{ snakeState.dir }}
  </code>
  <code class="text-center">
    {{ snakeState.coords.map((c) => `${c.x},${c.y}`) }}
  </code>
</template>

<style>
canvas {
  height: v-bind(CANVAS_HEIGHT_PX);
  width: v-bind(CANVAS_WIDTH_PX);
}
</style>
