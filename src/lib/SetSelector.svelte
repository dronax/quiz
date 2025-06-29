<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  function selectSet(setNumber) {
    dispatch('selectSet', setNumber);
  }
</script>

<div class="set-selector">
  <h2>Select a Question Set</h2>
  <div class="sets">
    {#each Array(10) as _, i}
      <button on:click={() => selectSet(i + 1)}>Set {i + 1}</button>
    {/each}
  </div>
</div>

<style>
  .set-selector {
    text-align: center;
  }

  .sets {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 1rem;
    margin-top: 1rem;
  }

  button {
    padding: 1rem 2rem;
    border: 1px solid var(--border-color);
    border-radius: 4px;
    background-color: var(--button-bg);
    cursor: pointer;
  }

  button:hover {
    background-color: var(--button-hover-bg);
  }
</style>
