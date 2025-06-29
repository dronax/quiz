<script lang="ts">
  import Quiz from './lib/Quiz.svelte';
  import SetSelector from './lib/SetSelector.svelte';

  let selectedSet = null;

  function handleSelectSet(event) {
    selectedSet = event.detail;
  }
</script>

<main>
  <h1>Nepal Engineering License Exam Quiz</h1>

  {#if selectedSet}
    <Quiz setNumber={selectedSet} />
  {:else}
    <SetSelector on:selectSet={handleSelectSet} />
  {/if}
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 2em;
    font-weight: 100;
  }
</style>
