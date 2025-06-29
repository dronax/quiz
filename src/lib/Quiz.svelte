<script>
  import { onMount } from 'svelte';

  export let setNumber;

  let questions = [];
  let currentQuestionIndex = 0;
  let selectedAnswer = null;
  let score = 0;
  let quizFinished = false;
  let showExplanation = false;

  onMount(async () => {
    const response = await fetch(`/src/lib/question-sets/set${setNumber}.json`);
    questions = await response.json();
  });

  function selectAnswer(option) {
    selectedAnswer = option;
    showExplanation = false; // Hide explanation when a new answer is selected
  }

  function submitAnswer() {
    if (selectedAnswer === questions[currentQuestionIndex].answer) {
      score += questions[currentQuestionIndex].marks;
    }
    showExplanation = true; // Show explanation after answering
  }

  function goToNextQuestion() {
    if (currentQuestionIndex < questions.length - 1) {
      currentQuestionIndex++;
      selectedAnswer = null;
      showExplanation = false;
    } else {
      quizFinished = true;
    }
  }

  function goToPreviousQuestion() {
    if (currentQuestionIndex > 0) {
      currentQuestionIndex--;
      selectedAnswer = null;
      showExplanation = false;
    }
  }
</script>

<div class="quiz-container">
  {#if quizFinished}
    <h2>Quiz Finished!</h2>
    <p>Your score: {score}/{questions.length}</p>
  {:else if questions.length > 0}
    {#if currentQuestionIndex > 0}
      <button class="back-arrow" on:click={goToPreviousQuestion}>
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" width="24px" height="24px">
          <path d="M0 0h24v24H0z" fill="none"/>
          <path d="M15.41 16.59L10.83 12l4.58-4.59L14 6l-6 6 6 6z"/>
        </svg>
      </button>
    {/if}
    <div class="question">
      <h2>{questions[currentQuestionIndex].question}</h2>
      <div class="options">
        {#each questions[currentQuestionIndex].options as option, i}
          <button
            class="option {selectedAnswer === String.fromCharCode(97 + i) ? 'selected' : ''}"
            on:click={() => selectAnswer(String.fromCharCode(97 + i))}
            disabled={showExplanation}
          >
            {String.fromCharCode(65 + i)}. {option}
          </button>
        {/each}
      </div>
      <div class="quiz-navigation">
        
        <button class="submit-btn" on:click={submitAnswer} disabled={!selectedAnswer || showExplanation}>Submit</button>
        {#if showExplanation && currentQuestionIndex < questions.length - 1}
          <button class="nav-btn" on:click={goToNextQuestion}>Next</button>
        {/if}
      </div>
    </div>
    <div class="explanation">
      {#if showExplanation}
        <p>{@html questions[currentQuestionIndex].explanation}</p>
      {/if}
    </div>
  {:else}
    <p>Loading questions...</p>
  {/if}
</div>

<style>
  .quiz-container {
    max-width: 600px;
    margin: 2rem auto;
    padding: 2rem;
    padding-top: 3rem; /* Added padding to make space for the arrow */
    border: 1px solid var(--border-color);
    border-radius: 8px;
    background-color: var(--container-bg);
    position: relative;
  }

  .back-arrow {
    position: absolute;
    top: 1rem;
    left: 1rem;
    background: none;
    border: none;
    cursor: pointer;
    padding: 0;
  }

  .back-arrow svg {
    color:var(--selected-color);
  }

  .question h2 {
    margin-bottom: 1rem;
  }

  .options {
    display: flex;
    flex-direction: column;
  }

  .option {
    margin: 0.5rem 0;
    padding: 1rem;
    border: 1px solid var(--border-color);
    border-radius: 4px;
    cursor: pointer;
    background-color: var(--button-bg);
  }

  .option:hover {
    background-color: var(--button-hover-bg);
  }

  .option.selected {
    background-color: var(--selected-bg);
    color: var(--selected-color);
  }

  .next-btn {
    margin-top: 1rem;
    padding: 0.8rem 1.5rem;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }

  .next-btn:disabled {
    background-color: #ccc;
    cursor: not-allowed;
  }

  .quiz-navigation {
    display: flex;
    justify-content: space-between;
    margin-top: 1rem;
  }

  .nav-btn,
  .submit-btn {
    padding: 0.8rem 1.5rem;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }

  .nav-btn:disabled,
  .submit-btn:disabled {
    background-color: #ccc;
    color: #666;
    cursor: not-allowed;
  }

  .explanation {
    margin-top: 1rem;
    padding: 1rem;
    border: 1px solid var(--border-color);
    background-color: var(--container-bg);
  }
</style>
