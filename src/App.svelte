<script lang="ts">
  import * as tome from "chromotome";

  import { onMount, onDestroy, setContext } from "svelte";
  import { writable } from "svelte/store";

  const width = writable(600);
  const height = writable(600);
  const pixelRatio = writable(window.devicePixelRatio);

  let canvas;
  let context: CanvasRenderingContext2D;
  let frame;

  const colors = tome.get("saami");

  let x = 0;
  let y = 0;

  let feedbackEl: HTMLDivElement;

  $: cols = 10;
  $: size = ($width * $pixelRatio) / cols;
  $: rows = Math.ceil(($height * $pixelRatio) / size);

  const s = (perc) => size * perc;
  const c = (perc) => perc * (Math.PI * 2);

  let fill = true;
  let autoplay = true;

  onMount(() => {
    // prepare canvas stores
    context = canvas.getContext("2d");
    handleResize();

    run();
  });

  function run() {
    x = 0;
    y = 0;

    console.log("hey?", cols, size);

    if (fill) {
      context.save();
      context.fillStyle = colors.background;
      context.fillRect(0, 0, $width * $pixelRatio, $height * $pixelRatio);
      context.restore();
    } else {
      context.clearRect(0, 0, $width * $pixelRatio, $height * $pixelRatio);
    }

    cancelAnimationFrame(frame);
    frame = requestAnimationFrame(update);
  }

  function update() {
    addDrawing();

    if (x < cols && y < rows && autoplay) {
      frame = requestAnimationFrame(update);
    }
  }

  function drawOne() {
    context.beginPath();
    context.moveTo(0, 0);
    context.lineTo(size, size);

    if (fill) {
      context.strokeStyle = colors.colors[0];
      context.stroke();
    } else {
      context.stroke();
    }
  }

  function drawTwo() {
    context.beginPath();
    context.arc(s(0.25), s(0.75), s(0.25), 0, c(1));

    if (fill) {
      context.fillStyle = colors.colors[1];
      context.fill();
    } else {
      context.stroke();
    }
  }

  function drawThree() {
    context.beginPath();
    context.rect(s(0.5), 0, s(0.5), s(0.5));

    if (fill) {
      context.fillStyle = colors.colors[2];
      context.fill();
    } else {
      context.stroke();
    }
  }

  function drawFour() {
    context.beginPath();
    context.arc(s(0.25), s(0.25), s(0.25), c(0.125), c(0.625));
    context.closePath();

    if (fill) {
      context.fillStyle = colors.colors[3];
      context.fill();
    } else {
      context.stroke();
    }
  }

  function drawFive() {
    context.beginPath();
    context.arc(s(0.75), s(0.75), s(0.25), c(0.625), c(0.125));
    context.closePath();

    if (fill) {
      context.fillStyle = colors.colors[4];
      context.fill();
    } else {
      context.stroke();
    }
  }

  function advance() {
    x = x + 1;

    if (x >= cols) {
      y = y + 1;
      x = 0;
    }
  }

  function clamp(val, min, max) {
    return Math.min(Math.max(val, min), max);
  }

  function handleResize() {
    canvas.style.setProperty(
      "--scale-down",
      clamp(window.innerWidth / ($width + 40), 0.5, 1)
    );
  }

  function addMessage(txt) {
    feedbackEl.innerText = txt;
  }

  function addDrawing() {
    const roll = Math.floor(Math.random() * 6) + 1;

    addMessage(roll);

    context.save();
    context.translate(x * size, y * size);
    context.lineWidth = size / 20;
    switch (roll) {
      case 1:
        drawOne();
        break;
      case 2:
        drawTwo();
        break;
      case 3:
        drawThree();
        break;
      case 4:
        drawFour();
        break;
      case 5:
        drawFive();
        break;
      case 6:
        advance();
        break;
    }
    context.restore();
  }

  function toggleAutoplay() {
    autoplay = !autoplay;
  }

  function restartCols(e) {
    cols = parseInt(e.target.value, 10);
    run();
  }
</script>

<main>
  <canvas
    bind:this={canvas}
    width={$width * $pixelRatio}
    height={$height * $pixelRatio}
    style="width: {$width}px; height: {$height}px;"
    on:click={addDrawing}
  />

  <div class="feedback">
    Last rolled: <span bind:this={feedbackEl} />
  </div>

  <div class="actions">
    <button on:click={toggleAutoplay}
      >Autoplay: {autoplay ? "on" : "off"}</button
    >
    <label>
      <button on:click={update} disabled={autoplay}>Next roll</button>
    </label>

    <label>
      <span>Columns ({cols})</span>
      <input
        type="range"
        min="5"
        max="100"
        value="10"
        on:change={restartCols}
      />
    </label>

    <label>
      <span>Fill?</span>
      <input type="checkbox" bind:checked={fill} />
    </label>

    <label>
      <button on:click={run}>Restart</button>
    </label>
  </div>
</main>

<svelte:window on:resize|passive={handleResize} />

<style>
  * {
    padding: 0;
    margin: 0;
  }

  main {
    display: grid;
    grid-template-columns: repeat(12, 1fr);

    max-width: 1400px;
    margin: 0 auto;

    overflow: hidden;

    align-items: center;
  }
  canvas {
    align-self: center;
    justify-self: center;

    grid-column: 1 / span 12;
    width: 100vw;

    transform: scale(var(--scale-down));
    transform-origin: top center;

    margin-bottom: calc((1 - var(--scale-down)) * (-100vw - 80px));
  }

  .feedback {
    display: flex;
    flex-direction: column;

    overflow: auto;

    grid-column: 1 / span 12;

    padding: 50px 0;

    font-size: 18px;
  }

  .feedback span {
    font-size: 32px;
  }

  .actions {
    grid-column: 1 / span 12;
  }

  .actions label {
    margin-bottom: 1em;
  }

  .actions label span {
    display: block;
  }

  @media screen and (min-width: 768px) {
    main {
      height: 100vh;
    }

    canvas {
      grid-row: 1 / span 2;
      grid-column: 1 / span 9;
    }

    .feedback,
    .actions {
      grid-column: 10 / span 3;
    }
  }

  @media screen and (min-width: 1000px) {
    canvas {
      grid-row: 1;
      grid-column: 1 / span 6;
    }

    .feedback {
      grid-column: 7 / span 2;
    }

    .actions {
      grid-column: 10 / span 2;
    }
  }
</style>
