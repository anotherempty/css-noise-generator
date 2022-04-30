<script>
  export let state = true;
</script>

{#if state}
  <slot name="true" />
{:else}
  <slot name="false" />
{/if}
