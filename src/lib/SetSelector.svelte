<script>
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  function selectSet(setNumber) {
    dispatch('selectSet', setNumber);
  }
</script>

<div class="set-selector fade-in">
  <div class="selector-header">
    <h2 class="selector-title">Choose Your Question Set</h2>
    <p class="selector-description">Each set contains carefully curated questions to test your engineering knowledge</p>
  </div>
  
  <div class="sets-grid">
    {#each Array(10) as _, i}
      <button 
        class="set-card btn btn-secondary" 
        on:click={() => selectSet(i + 1)}
      >
        <div class="set-number">Set {i + 1}</div>
        <div class="set-info">
          <span class="question-count">30 Questions</span>
          <span class="time-limit">60 Minutes</span>
        </div>
        <div class="set-icon">📚</div>
      </button>
    {/each}
  </div>

  <div class="info-cards">
    <div class="info-card">
      <div class="info-icon">⏱️</div>
      <h3>Time Limit</h3>
      <p>60 minutes per quiz set</p>
    </div>
    <div class="info-card">
      <div class="info-icon">📊</div>
      <h3>Scoring</h3>
      <p>Instant feedback and explanations</p>
    </div>
    <div class="info-card">
      <div class="info-icon">🎯</div>
      <h3>Practice</h3>
      <p>Real exam-style questions</p>
    </div>
  </div>
</div>

<style>
  .set-selector {
    width: 100%;
    max-width: 900px;
    margin: 0 auto;
    padding: 2rem;
  }

  .selector-header {
    text-align: center;
    margin-bottom: 3rem;
  }

  .selector-title {
    font-size: 2rem;
    font-weight: 600;
    color: var(--text-primary);
    margin-bottom: 0.75rem;
  }

  .selector-description {
    font-size: 1rem;
    color: var(--text-secondary);
    max-width: 500px;
    margin: 0 auto;
  }

  .sets-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1.5rem;
    margin-bottom: 4rem;
  }

  .set-card {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 2rem 1.5rem;
    min-height: 160px;
    position: relative;
    transition: all 0.3s ease-in-out;
    border: 2px solid var(--border-color);
    background: var(--bg-primary);
  }

  .set-card:hover {
    border-color: var(--primary-color);
    background: var(--primary-light);
    transform: translateY(-4px);
    box-shadow: var(--shadow-xl);
  }

  .set-number {
    font-size: 1.25rem;
    font-weight: 600;
    color: var(--text-primary);
    margin-bottom: 0.75rem;
  }

  .set-info {
    display: flex;
    flex-direction: column;
    gap: 0.25rem;
    text-align: center;
    margin-bottom: 1rem;
  }

  .question-count,
  .time-limit {
    font-size: 0.875rem;
    color: var(--text-secondary);
  }

  .set-icon {
    font-size: 1.5rem;
    opacity: 0.7;
    transition: all 0.3s ease-in-out;
  }

  .set-card:hover .set-icon {
    opacity: 1;
    transform: scale(1.1);
  }

  .info-cards {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 1.5rem;
    margin-top: 2rem;
  }

  .info-card {
    background: var(--bg-primary);
    border-radius: var(--radius-lg);
    padding: 2rem;
    text-align: center;
    border: 1px solid var(--border-color);
    transition: all 0.3s ease-in-out;
  }

  .info-card:hover {
    transform: translateY(-2px);
    box-shadow: var(--shadow-md);
  }

  .info-icon {
    font-size: 2rem;
    margin-bottom: 1rem;
    display: block;
  }

  .info-card h3 {
    font-size: 1.125rem;
    font-weight: 600;
    color: var(--text-primary);
    margin-bottom: 0.5rem;
  }

  .info-card p {
    font-size: 0.875rem;
    color: var(--text-secondary);
  }

  @media (max-width: 768px) {
    .set-selector {
      padding: 1rem;
    }

    .sets-grid {
      grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
      gap: 1rem;
    }

    .set-card {
      padding: 1.5rem 1rem;
      min-height: 140px;
    }

    .selector-title {
      font-size: 1.5rem;
    }

    .info-cards {
      grid-template-columns: 1fr;
      gap: 1rem;
    }

    .info-card {
      padding: 1.5rem;
    }
  }
</style>