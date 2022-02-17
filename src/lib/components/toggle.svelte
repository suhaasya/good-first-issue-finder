<script lang="ts">
  export let checked: boolean;
  export let labelLeft: string;
  export let labelRight: string;
  export let id: string;
</script>

<div
  class="inline-flex items-center justify-center space-x-1 rounded-full bg-white px-2 py-2 shadow-standard transition-all duration-200"
>
  <label
    class:inversed={!checked}
    class=" cursor-pointer rounded-full px-2 font-semibold transition-all duration-200"
    for={id}>{@html labelLeft}</label
  >
  <div><input on:change {id} bind:checked type="checkbox" class="hidden" /></div>
  <label
    class:inversed={checked}
    class="inversed cursor-pointer rounded-full px-2 font-semibold transition-all duration-200"
    for={id}>{@html labelRight}</label
  >
</div>

<style lang="postcss">
  .inversed {
    @apply bg-eddiehub-200 text-white;
  }
</style>
