<script lang="ts">
  import * as tome from "chromotome";

  import { onMount, onDestroy, setContext } from "svelte";
  import { writable } from "svelte/store";

  // Pretty arbitrary width/height
  const width = writable(600);
  const height = writable(600);
  const pixelRatio = writable(window.devicePixelRatio);

  let canvas;
  let context: CanvasRenderingContext2D;
  let frame;

  // Makes the palette out of the lovely chromotome library
  const colors = tome.get("saami");

  // cell position
  let x = 0;
  let y = 0;

  // Store a reference to the div that will show the dice roll
  let feedbackEl: HTMLDivElement;

  // Reactive variables to deal with the cells and their sizes
  $: cols = 10;
  $: size = ($width * $pixelRatio) / cols;
  $: rows = Math.ceil(($height * $pixelRatio) / size);

  // Little helper functions for size/circle circumference
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

  /**
   * This is the function that kicks off the drawing
   */
  function run() {
    x = 0;
    y = 0;

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

  // Animation loop
  function update() {
    addDrawing();

    if (x < cols && y < rows && autoplay) {
      frame = requestAnimationFrame(update);
    }
  }

  // Draws the diagonal line
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

  // Draws the circle bottom left
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

  // Draws the square top right
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

  // Draws the half circle top left
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

  // Draws the half circle bottom right
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

  // Utility to help with the advancing of cells
  function advance() {
    x = x + 1;

    if (x >= cols) {
      y = y + 1;
      x = 0;
    }
  }

  // Clamp a value, wish this was just in javascript
  function clamp(val, min, max) {
    return Math.min(Math.max(val, min), max);
  }

  // This is just for the visual on mobile, so the canvas scales with the device a bit
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
    // Simulates 'rolling' a 6 sided dice
    const roll = Math.floor(Math.random() * 6) + 1;

    addMessage(roll);

    // Super simple switch case to draw after the dice roll
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
    // It gets funky if we dont restart the drawing after changing the columns
    cols = parseInt(e.target.value, 10);
    run();
  }
</script>

<main>
  <div class="banner">
    <div class="left">
      <a href="https://github.com/jthawme/handmade-generative" target="_blank"
        >Site code (made with Svelte)</a
      >
      <a href="https://jthaw.me/files/activity-sheet-2.pdf" target="_blank"
        >Download PDF</a
      >
    </div>

    <div class="right">
      <a href="https://jthaw.me/newsletter"
        >This was made for my newsletter. Sign up for more like it</a
      >
    </div>
  </div>
  <div class="wrapper">
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
  </div>
</main>

<svelte:window on:resize|passive={handleResize} />

<style>
  main {
    display: flex;
    flex-direction: column;

    overflow: hidden;

    min-height: 100vh;
  }

  .wrapper {
    display: grid;
    grid-template-columns: repeat(12, 1fr);

    max-width: 1400px;
    gap: 10px;
    margin: 0 auto;

    flex-grow: 1;

    align-items: center;
  }

  .banner {
    display: flex;

    background-color: #e7e6e4;
    flex-direction: column;

    margin-bottom: 2em;
  }

  .left {
    display: flex;
    flex-direction: column;
    padding: 10px;

    flex-grow: 1;
  }

  .left a {
    color: #333;
    margin-right: 20px;
    margin-bottom: 10px;
  }

  .right a {
    display: flex;

    align-items: center;
    color: white;
    background-color: #39638f;
    padding: 10px 20px;
  }

  .right a:hover {
    background-color: #e3b83e;
    text-decoration: none;
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
    .banner {
      margin-bottom: 5px;
    }

    .banner,
    .left {
      flex-direction: row;
    }

    .left a {
      margin-bottom: 0;
    }

    main {
      height: 100vh;
    }

    .wrapper {
      grid-template-columns: 600px 1fr;
      width: 100%;
    }

    canvas {
      grid-row: 1 / span 2;
      grid-column: 1;
    }

    .feedback,
    .actions {
      grid-column-end: span 1;
    }

    .feedback,
    .actions {
      grid-column-start: 2;
    }
  }

  @media screen and (min-width: 1000px) {
    canvas {
      grid-row: 1;
    }

    .wrapper {
      gap: 30px;
      grid-template-columns: 600px 1fr 1fr;
    }

    .feedback,
    .actions {
      grid-column-start: unset;
      grid-column-end: span 1;
    }
  }
</style>
