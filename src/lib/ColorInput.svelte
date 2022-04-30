<script>
  export let value = "#ffffff";
</script>

<div class="flex gap-2">
  <div class="flex flex-1 items-center">
    <input type="text" value={value.slice(1)} readonly />
  </div>
  <input type="color" bind:value />
</div>

<style>
  input[type="text"] {
    appearance: none;
    border-radius: 8px;
    background: var(--color-1);
    border: none;
    padding: 0 4px;
    height: 2em;
    width: 100%;
  }

  input[type="color"] {
    width: 64px;
    background: var(--color-1);
    border: none;
    height: 2em;
    padding: 2px 4px;
    border-radius: 8px;
  }
</style>
