<script>
  /**
   * TODO :
   * - refactor this mess
   * - add tests
   */

  import { onMount } from "svelte";

  import Slider from "./lib/Slider.svelte";
  import HueSelector from "./lib/HueSelector.svelte";
  import ColorInput from "./lib/ColorInput.svelte";
  import SwitchBtn from "./lib/SwitchBtn.svelte";

  let canvas = document.createElement("canvas");
  let ctx = canvas.getContext("2d");

  let previewer;

  let url = "";
  let css = "";

  let patternWidth = 48;
  let minWidth = 4;
  let maxWidth = 300;

  let patternHeight = 48;
  let minHeight = 4;
  let maxHeight = 300;

  let grainOpacity = 0.1;
  let minOpacity = 0.01;
  let maxOpacity = 0.9;

  let grainDensity = 1;
  let minDensity = 1;
  let maxDensity = 25;

  let grainColor = 0;
  let minColor = 0;
  let maxColor = 360;

  let bgColor = "#ffffff";

  let bnw = true;

  let showHueSelector = false;

  let currentTheme = "light";
  let bgPreview = true;

  $: bnw = !showHueSelector;

  const init = () => {
    patternWidth = 48;

    patternHeight = 48;

    grainOpacity = 0.1;

    grainDensity = 1;

    grainColor = 0;

    if (currentTheme == "dark") bgColor = "#1e1e1e";
    else if (currentTheme == "light") bgColor = "#fdf6e3";
  };

  const randomize = () => {
    patternWidth = Math.floor(random(minWidth, maxWidth));

    patternHeight = Math.floor(random(minHeight, maxHeight));

    grainOpacity = parseFloat(random(minOpacity, maxOpacity).toFixed(2));

    grainDensity = Math.floor(random(minDensity, maxDensity));

    if (!bnw) grainColor = Math.floor(random(minColor, maxColor));

    bgColor =
      "#" + ((Math.random() * 0xffffff) << 0).toString(16).padStart(6, "0");
  };

  const random = (min, max) => {
    return Math.random() * (max - min) + min;
  };

  let makeNoise = () => {
    canvas.width = patternWidth;
    canvas.height = patternHeight;

    let lightness;
    let saturation;

    if (bnw) saturation = 0;
    else saturation = 100;

    let x;
    let y;

    let opacity;

    for (x = 0; x < patternWidth; x += grainDensity) {
      for (y = 0; y < patternHeight; y += grainDensity) {
        lightness = random(0, 100);
        opacity = parseFloat(
          random(minOpacity, (grainOpacity + minOpacity) / 2).toFixed(2)
        );

        ctx.fillStyle = `hsla(${grainColor}, ${saturation}%, ${lightness}%, ${opacity})`;
        ctx.fillRect(x, y, grainDensity, grainDensity);
      }
    }
    url = canvas.toDataURL("image/png");
  };

  const setBackgrounds = () => {
    if (bgPreview) {
      document.body.style.backgroundImage = "url(" + url + ")";
    } else {
      document.body.style.backgroundImage = "none";
    }
    if (previewer !== undefined) {
      previewer.style.backgroundImage = `url(${url})`;
    }
  };

  $: bnw,
    grainColor,
    patternWidth,
    patternHeight,
    grainOpacity,
    grainDensity && makeNoise();

  $: bgPreview, url && setBackgrounds();

  $: css = `
    background-color: ${bgColor};
    background-image: url(${url});
  `;

  const setTheme = (theme) => {
    document.body.classList.remove(currentTheme);
    currentTheme = theme;
    document.body.classList.add(currentTheme);
  };

  onMount(() => {
    if (window.matchMedia) {
      if (window.matchMedia("(prefers-color-scheme: dark)").matches) {
        currentTheme = "dark";
      } else {
        currentTheme = "light";
      }
    } else {
      currentTheme = "light";
    }

    if (currentTheme == "dark") bgColor = "#1e1e1e";
    else if (currentTheme == "light") bgColor = "#fdf6e3";
    setTheme(currentTheme);

    makeNoise();
    setBackgrounds();
  });

  window
    .matchMedia("(prefers-color-scheme: dark)")
    .addEventListener("change", (event) => {
      setTheme(event.matches ? "dark" : "light");
    });
</script>

<div class="flex gap-4 justify-end txt-default py-2">
  <SwitchBtn state={currentTheme == "dark" ? true : false}>
    <button
      slot="true"
      class="icon-button"
      on:click={() => setTheme(currentTheme == "dark" ? "light" : "dark")}
      title="Dark mode"
    >
      <span class="material-icons-outlined md-36"> light_mode </span>
    </button>
    <button
      slot="false"
      class="icon-button"
      on:click={() => setTheme(currentTheme == "dark" ? "light" : "dark")}
      title="Light mode"
    >
      <span class="material-icons-outlined md-36"> dark_mode </span>
    </button>
  </SwitchBtn>
  <SwitchBtn state={bgPreview}>
    <button
      slot="true"
      class="icon-button"
      on:click={() => (bgPreview = !bgPreview)}
      title="Remove background preview"
      ><span class="material-icons-outlined md-36">
        format_color_reset
      </span></button
    >
    <button
      slot="false"
      class="icon-button"
      on:click={() => (bgPreview = !bgPreview)}
      title="Apply on background"
      ><span class="material-icons-outlined md-36"> format_paint </span></button
    >
  </SwitchBtn>
</div>
<main
  class="bg-color-2 txt-default p-10 rounded-3xl shadow-2xl transition-colors ease-out duration-500"
>
  <h1 class="pb-4 text-xl font-bold">CSS Noise Texture Generator</h1>
  <div class="flex gap-10 mb-4">
    <div class="flex flex-col gap-4">
      <div
        bind:this={previewer}
        class="bg-gray-600 relative rounded-lg shadow-lg"
        style="width:300px; height:300px; background-color:{bgColor};"
      >
        <div
          class="absolute border border-black rounded-lg"
          style="width:{patternWidth}px; height:{patternHeight}px"
        />
      </div>

      <div>
        Background Color Preview
        <ColorInput bind:value={bgColor} />
      </div>
    </div>
    <div class="flex flex-col gap-2 flex-1">
      <div>
        <Slider min={minWidth} max={maxWidth} bind:value={patternWidth} step={1}
          >Pattern Width</Slider
        >
      </div>
      <div>
        <Slider
          min={minHeight}
          max={maxHeight}
          bind:value={patternHeight}
          step="1">Pattern Height</Slider
        >
      </div>
      <div>
        <Slider
          min={minOpacity}
          max={maxOpacity}
          bind:value={grainOpacity}
          step="0.01">Noise Opacity</Slider
        >
      </div>
      <div>
        <Slider
          min={minDensity}
          max={maxDensity}
          bind:value={grainDensity}
          step="1">Noise Density</Slider
        >
      </div>
      <div>
        <div class="flex items-center gap-2 py-2">
          <label for="noise-color">Noise Color</label>
          <input
            type="checkbox"
            name="noise-color"
            bind:checked={showHueSelector}
          />
        </div>
        {#if showHueSelector}
          <HueSelector min={minColor} max={maxColor} bind:value={grainColor} />
        {/if}
      </div>
      <div class="flex gap-2 justify-end pt-4">
        <button class="button" on:click={() => init()}>
          <span class="material-icons-outlined"> restart_alt </span>
          Reset</button
        >
        <button class="button" on:click={() => randomize()}>
          <span class="material-icons-outlined"> question_mark </span>
          Random</button
        >
      </div>
    </div>
  </div>
  <div class="rounded-2xl bg-color-1">
    <div class="flex gap-2 p-4 justify-end">
      <a
        class="icon-button"
        href={url}
        download="noise-bg.png"
        title="Download"
      >
        <span class="material-icons-outlined"> file_download </span>
      </a>
      <button class="icon-button" title="Copy CSS"
        ><span class="material-icons-outlined"> content_copy </span></button
      >
    </div>
    <div class="px-4 pb-4 break-words text-sm">
      {css}
    </div>
  </div>
</main>

<style lang="postcss" global>
  @tailwind base;
  @tailwind components;
  @tailwind utilities;

  html {
    padding: 0;
    margin: 0;
  }
  body {
    @apply px-10;
    @apply bg-color-1;
  }

  :root {
    --color-1: black;
    --color-2: black;
    --color-3: black;
    --color-4: black;
    --text-color: black;
    --text-secondary: black;
  }

  .dark {
    --color-1: #1e1e1e;
    --color-2: #272727;
    --color-3: #353535;
    --color-4: #505050;
    --text-color: #848484;
    --text-secondary: #505050;
  }

  .light {
    --color-1: #fdf6e3;
    --color-2: #dbd5c1;
    --color-3: #c9c3b3;
    --color-4: #beb7a5;
    --text-color: #6e6d6b;
    --text-secondary: #a79e83;
  }

  .button {
    @apply pr-4;
    @apply py-1;
    @apply rounded;
    @apply appearance-none;
    @apply no-underline;
    @apply border-0;
    @apply bg-color-3;
    @apply txt-default;
    @apply flex;
  }
  .button:hover {
    @apply no-underline;
  }
  .button > span {
    @apply mx-2;
  }

  .icon-button {
    @apply flex;
    @apply appearance-none;
    @apply border-0;
    @apply txt-secondary;
  }
  .icon-button:hover {
    @apply txt-default;
  }

  .bg-color-1 {
    background-color: var(--color-1);
  }
  .bg-color-2 {
    background-color: var(--color-2);
  }
  .bg-color-3 {
    background-color: var(--color-3);
  }
  .bg-color-4 {
    background-color: var(--color-4);
  }
  .txt-default {
    color: var(--text-color);
  }
  .txt-secondary {
    color: var(--text-secondary);
  }

  ::-webkit-scrollbar {
    width: 16px;
  }
  ::-webkit-scrollbar-track {
    background-color: var(--color-1);
  }
  ::-webkit-scrollbar-thumb {
    background-color: var(--color-2);
    border: 4px solid var(--color-1);
    border-radius: 10px;
  }
  ::-webkit-scrollbar-thumb:hover {
    background-color: var(--color-3);
  }

  input[type="checkbox"] {
    background-color: var(--color-1);
    width: 2em;
    height: 2em;
    border-radius: 8px;
    border: none;
    appearance: none;
    position: relative;
  }
  input[type="checkbox"]:focus {
    outline: 1px solid var(--text-color);
  }
  input[type="checkbox"]::after {
    content: "";
    position: absolute;
    width: 0.1em;
    height: 0.1em;
    box-sizing: border-box;
    opacity: 0;
    left: 50%;
    top: 50%;
    transform: translate3D(-50%, -50%, 0);
    background-color: var(--color-2);
    border-radius: 40px;
    transition-property: width, height, opacity;
    transition: 200ms ease-out;
    z-index: 3;
  }

  input[type="checkbox"]:checked::after {
    width: 1.4em;
    height: 1.4em;
    opacity: 1;
    border-radius: 4px;
  }
  input[type="checkbox"]:checked:hover::after {
    background-color: var(--color-3);
  }

  input[type="checkbox"]::before {
    content: "";
    position: absolute;
    width: 1.4em;
    height: 1.4em;
    opacity: 0;
    left: 50%;
    top: 50%;
    transform: translate3D(-50%, -50%, 0);
    background-color: var(--color-2);
    border-radius: 4px;
    transition-property: width, height, opacity;
    transition: 250ms ease-in-out;
    z-index: 2;
  }
  input[type="checkbox"]:hover::before {
    opacity: 1;
  }

  .material-icons-outlined.md-18 {
    font-size: 18px;
  }
  .material-icons-outlined.md-24 {
    font-size: 24px;
  }
  .material-icons-outlined.md-36 {
    font-size: 36px;
  }
  .material-icons-outlined.md-48 {
    font-size: 48px;
  }
</style>
