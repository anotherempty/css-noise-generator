<script>
  export let min = 0;
  export let max = 1;
  export let value = 1;
  export let step = 1;
</script>

<slot />
<div class="flex gap-2">
  <div class="flex flex-1 items-center">
    <input type="range" {min} {max} bind:value {step} />
  </div>
  <input type="number" {min} {max} bind:value {step} />
</div>

<style>
  input[type="number"] {
    appearance: none;
    border-radius: 8px;
    background: var(--color-1);
    border: none;
    padding: 0 4px;
    width: 64px;
  }
  input[type="number"]:focus{
    outline: 1px solid var(--text-color);
  }

  input[type="range"] {
    appearance: none;
    width: 100%;
    height: 2em;
    background: var(--color-1);
    outline: none;
    border-radius: 8px;
    border: none;
    padding: 0 4px;
  }

  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 12px;
    height: 1.6em;
    background: var(--color-2);
    cursor: pointer;
    border-radius: 8px;
  }
  input[type="range"]::-webkit-slider-thumb:hover{
    background: var(--color-3);
  }
  input[type="range"]::-webkit-slider-thumb:focus{
    border: 1px solid var(--color-4);
  }
  input[type="range"]::-webkit-slider-thumb:active{
    border: 1px solid var(--color-4);
  }

  input[type="range"]::-moz-range-thumb {
    appearance: none;
    width: 12px;
    height: 18px;
    background: transparent;
    cursor: pointer;
    border: 1px solid white;
    border-radius: 8px;
  }
</style>
