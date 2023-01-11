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

let playing;
let context;
const canvas = ref(null);

// prettier-ignore
const spawnPos = {
  x: 100,
  y: 46
};
const spawnPosCenter = {
  x: CANVAS_WIDTH / PIXEL_SIZE / SQUARE_SIZE,
  y: CANVAS_HEIGHT / PIXEL_SIZE / SQUARE_SIZE,
};

const initleftdirstate = {
  coords: [
    spawnPos,
    { ...spawnPos, x: spawnPos.x + SQUARE_SIZE },
    { ...spawnPos, x: spawnPos.x + 2 * SQUARE_SIZE },
    { ...spawnPos, x: spawnPos.x + 3 * SQUARE_SIZE },
  ],
  dir: DIR_LEFT,
};

const initrightdirstate = {
  coords: [
    spawnPos,
    { ...spawnPos, x: spawnPos.x - SQUARE_SIZE },
    { ...spawnPos, x: spawnPos.x - 2 * SQUARE_SIZE },
    { ...spawnPos, x: spawnPos.x - 3 * SQUARE_SIZE },
  ],
  dir: DIR_RIGHT,
};

const initupdirstate = {
  coords: [
    spawnPos,
    { ...spawnPos, y: spawnPos.y + SQUARE_SIZE },
    { ...spawnPos, y: spawnPos.y + 2 * SQUARE_SIZE },
    { ...spawnPos, y: spawnPos.y + 3 * SQUARE_SIZE },
  ],
  dir: DIR_UP,
};

const initdowndirstate = {
  coords: [
    spawnPos,
    { ...spawnPos, y: spawnPos.y - SQUARE_SIZE },
    { ...spawnPos, y: spawnPos.y - 2 * SQUARE_SIZE },
    { ...spawnPos, y: spawnPos.y - 3 * SQUARE_SIZE },
  ],
  dir: DIR_DOWN,
};

const snakeState = ref(initdowndirstate);
// const snakeState = initdowndirstate;
const latestKeydown = ref(null);

onMounted(() => {
  context = canvas.value.getContext("2d", {
    alpha: false,
  });

  fillBg("#4F822B");

  drawSnake();

  playing = setInterval(onTick, TICKRATE);
  // drawGrid({
  //   drawFirstLine: true,
  //   color: "#4F822B88",
  // });
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

const drawGreenSquare = (x, y, striped) => {
  if (striped) {
    if (snakeState.value.dir === DIR_LEFT) {
      /// ONE SQUARE SLANT = \
      // bot left
      drawGreenPx(x, y + 2);
      // top right
      drawGreenPx(x + 1, y + 1);
    } else if (snakeState.value.dir === DIR_RIGHT) {
      /// ONE SQUARE SLANT = /
      // bot right
      drawGreenPx(x + 1, y + 2);
      // top left
      drawGreenPx(x, y + 1);
    }
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

  const { coords, dir } = snakeState.value;
  const len = coords.length;
  const head = coords[0];
  const tail = coords[len - 1];

  /// HEAD
  // MOUTH
  drawGreenSquare(head.x, head.y);
  // EYE
  drawEye(head);
  //// BODY
  const stripeFrequency = 2;
  coords.slice(1, len - 1).forEach((coord, i) => {
    if (DIR_UP === dir) {
      drawGreenSquare(wrapxc(coord.x), wrapyc(coord.y), !(i % stripeFrequency));
    } else if (DIR_DOWN === dir) {
      drawGreenSquare(wrapxc(coord.x), wrapyc(coord.y), !(i % stripeFrequency));
    } else if (DIR_LEFT === dir) {
      drawGreenSquare(wrapxc(coord.x + SQUARE_SIZE), wrapyc(coord.y), !(i % stripeFrequency));
    } else if (DIR_RIGHT === dir) {
      drawGreenSquare(wrapxc(coord.x - SQUARE_SIZE), wrapyc(coord.y), !(i % stripeFrequency));
    }
  });

  /// TAIL
  // ONE SEGMENT (STRIPED)
  if (dir === DIR_UP) {
    drawGreenPx(wrapxc(tail.x), wrapyc(tail.y));
  } else if (dir === DIR_LEFT) {
    drawGreenPx(wrapxc(tail.x + 2), wrapyc(tail.y + 2));
    drawGreenPx(wrapxc(tail.x + 3), wrapyc(tail.y + 1));
    drawGreenPx(wrapxc(tail.x + 4), wrapyc(tail.y + 2));
    drawGreenPx(wrapxc(tail.x + 5), wrapyc(tail.y + 2));
  } else if (dir === DIR_RIGHT) {
    drawGreenPx(wrapxc(tail.x - 2 * SQUARE_SIZE + 3), wrapyc(tail.y + 2));
    drawGreenPx(wrapxc(tail.x - 2 * SQUARE_SIZE + 2), wrapyc(tail.y + 1));
    drawGreenPx(wrapxc(tail.x - 2 * SQUARE_SIZE + 1), wrapyc(tail.y + 2));
    drawGreenPx(wrapxc(tail.x - 2 * SQUARE_SIZE), wrapyc(tail.y + 2));
  }

  function drawEye(head) {
    switch (dir) {
      case DIR_UP:
        drawGreenPx(head.x + 1, wrapyc(head.y + SQUARE_SIZE + 1));
        drawGreenPx(head.x - 1, wrapyc(head.y + SQUARE_SIZE + 1));
        drawGreenPx(head.x, wrapyc(head.y + SQUARE_SIZE + 2));
        drawGreenPx(head.x + 1, wrapyc(head.y + SQUARE_SIZE + 2));
        break;
      case DIR_DOWN:
        drawGreenPx(head.x, wrapyc(head.y));
        drawGreenPx(head.x + 2, wrapyc(head.y));
        drawGreenPx(head.x, wrapyc(head.y - 1));
        drawGreenPx(head.x + 1, wrapyc(head.y - 1));
        break;
      case DIR_LEFT:
        drawGreenPx(wrapxc(head.x + SQUARE_SIZE), head.y);
        drawGreenPx(wrapxc(head.x + SQUARE_SIZE), head.y + 2);
        drawGreenPx(wrapxc(head.x + SQUARE_SIZE + 1), head.y + 1);
        drawGreenPx(wrapxc(head.x + SQUARE_SIZE + 1), head.y + 2);
        break;
      case DIR_RIGHT:
        drawGreenPx(wrapxc(head.x - SQUARE_SIZE), head.y + 1);
        drawGreenPx(wrapxc(head.x - SQUARE_SIZE), head.y + 2);
        drawGreenPx(wrapxc(head.x - SQUARE_SIZE + 1), head.y);
        drawGreenPx(wrapxc(head.x - SQUARE_SIZE + 1), head.y + 2);
        break;
      default:
        console.debug("default");
    }
  }
};

function unshiftNewHeadCoord() {
  const currentHead = snakeState.value.coords[0];
  switch (snakeState.value.dir) {
    case DIR_UP:
      snakeState.value.coords.unshift({
        x: currentHead.x,
        y: currentHead.y > 0 ? currentHead.y - SQUARE_SIZE : CANVAS_HEIGHT / PIXEL_SIZE - SQUARE_SIZE,
      });
      break;
    case DIR_DOWN:
      snakeState.value.coords.unshift({
        x: currentHead.x,
        y: currentHead.y > 0 ? currentHead.y + SQUARE_SIZE : CANVAS_HEIGHT / PIXEL_SIZE + SQUARE_SIZE,
      });
      break;
    case DIR_LEFT:
      snakeState.value.coords.unshift({
        x: currentHead.x > 0 ? currentHead.x - SQUARE_SIZE : CANVAS_WIDTH / PIXEL_SIZE - SQUARE_SIZE,
        y: currentHead.y,
      });
      break;
    case DIR_RIGHT:
      snakeState.value.coords.unshift({
        x: currentHead.x + SQUARE_SIZE < CANVAS_WIDTH / PIXEL_SIZE ? currentHead.x + SQUARE_SIZE : 0,
        y: currentHead.y,
      });
      break;
  }
}

const drawGreenPx = (x, y) => {
  context.fillStyle = "#213404";
  context.fillRect(x * PIXEL_SIZE, y * PIXEL_SIZE, PIXEL_SIZE, PIXEL_SIZE);
};

const drawRedPx = (x, y) => {
  context.fillStyle = "red";
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
  snakeState.value.coords = initupdirstate.coords;
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
  console.log("invoke onkeydown");
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
    class="mi-auto border border-white"
  />
  <button @click="reset">reset</button>
  <button @click="togglepause">{{ playing ? "pause" : "resume" }}</button>
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
