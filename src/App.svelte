<script lang="ts">
  import Quiz from './lib/Quiz.svelte';
  import SetSelector from './lib/SetSelector.svelte';

  let selectedSet = null;

  function handleSelectSet(event) {
    selectedSet = event.detail;
  }

  function handleBackToSets() {
    selectedSet = null;
  }
</script>

<main class="main-container">
  {#if !selectedSet}
    <div class="header">
      <h1 class="title">
        <span class="title-icon">🎓</span>
        Nepal Engineering License Exam Quiz
      </h1>
      <p class="subtitle">Test your knowledge and prepare for success</p>
    </div>
  {/if}

  <div class="content">
    {#if selectedSet}
      <Quiz setNumber={selectedSet} on:backToSets={handleBackToSets} />
    {:else}
      <SetSelector on:selectSet={handleSelectSet} />
    {/if}
  </div>
</main>

<style>
  .main-container {
    max-width: 1200px;
    margin: 0 auto;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }

  .header {
    text-align: center;
    margin-bottom: 3rem;
    padding: 2rem 1rem;
  }

  .title {
    font-size: clamp(2rem, 5vw, 3.5rem);
    font-weight: 700;
    background: linear-gradient(135deg, var(--primary-color), #8b5cf6);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    margin-bottom: 1rem;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    line-height: 1.2;
  }

  .title-icon {
    font-size: 0.8em;
    filter: drop-shadow(0 4px 8px rgba(59, 130, 246, 0.3));
  }

  .subtitle {
    font-size: 1.125rem;
    color: var(--text-secondary);
    font-weight: 400;
    max-width: 600px;
    margin: 0 auto;
  }

  .content {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: flex-start;
  }

  @media (max-width: 768px) {
    .header {
      margin-bottom: 2rem;
      padding: 1rem;
    }

    .title {
      flex-direction: column;
      gap: 0.5rem;
    }

    .subtitle {
      font-size: 1rem;
    }
  }
</style>