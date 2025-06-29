<script>
  import { onMount, createEventDispatcher } from 'svelte';

  export let setNumber;
  
  const dispatch = createEventDispatcher();

  let questions = [];
  let currentQuestionIndex = 0;
  let selectedAnswer = null;
  let score = 0;
  let quizFinished = false;
  let showExplanation = false;
  let timeRemaining = 60 * 60; // 60 minutes in seconds
  let timerInterval;
  let quizStarted = false;

  $: minutes = Math.floor(timeRemaining / 60);
  $: seconds = timeRemaining % 60;
  $: isTimeRunningOut = timeRemaining <= 300; // Last 5 minutes
  $: isCriticalTime = timeRemaining <= 60; // Last minute

  onMount(async () => {
    const response = await fetch(`/src/lib/question-sets/set${setNumber}.json`);
    questions = await response.json();
    startTimer();
  });

  function startTimer() {
    quizStarted = true;
    timerInterval = setInterval(() => {
      if (timeRemaining > 0 && !quizFinished) {
        timeRemaining--;
      } else {
        finishQuiz();
      }
    }, 1000);
  }

  function finishQuiz() {
    quizFinished = true;
    if (timerInterval) {
      clearInterval(timerInterval);
    }
  }

  function selectAnswer(option) {
    selectedAnswer = option;
    showExplanation = false;
  }

  function submitAnswer() {
    if (selectedAnswer === questions[currentQuestionIndex].answer) {
      score += questions[currentQuestionIndex].marks;
    }
    showExplanation = true;
  }

  function goToNextQuestion() {
    if (currentQuestionIndex < questions.length - 1) {
      currentQuestionIndex++;
      selectedAnswer = null;
      showExplanation = false;
    } else {
      finishQuiz();
    }
  }

  function goToPreviousQuestion() {
    if (currentQuestionIndex > 0) {
      currentQuestionIndex--;
      selectedAnswer = null;
      showExplanation = false;
    }
  }

  function backToSets() {
    if (timerInterval) {
      clearInterval(timerInterval);
    }
    dispatch('backToSets');
  }

  function formatTime(time) {
    return time.toString().padStart(2, '0');
  }
</script>

<div class="quiz-container fade-in">
  {#if quizFinished}
    <div class="quiz-results card">
      <div class="results-header">
        <div class="results-icon">🎉</div>
        <h2>Quiz Completed!</h2>
        <p class="results-subtitle">
          {#if timeRemaining <= 0}
            Time's up! Here are your results:
          {:else}
            Great job! Here are your results:
          {/if}
        </p>
      </div>
      
      <div class="score-display">
        <div class="score-circle">
          <span class="score-number">{score}</span>
          <span class="score-total">/{questions.reduce((sum, q) => sum + q.marks, 0)}</span>
        </div>
        <p class="score-label">Total Score</p>
      </div>

      <div class="quiz-stats">
        <div class="stat">
          <span class="stat-value">{questions.length}</span>
          <span class="stat-label">Questions</span>
        </div>
        <div class="stat">
          <span class="stat-value">{Math.floor((60 * 60 - timeRemaining) / 60)}</span>
          <span class="stat-label">Minutes Used</span>
        </div>
        <div class="stat">
          <span class="stat-value">{Math.round((score / questions.reduce((sum, q) => sum + q.marks, 0)) * 100)}%</span>
          <span class="stat-label">Accuracy</span>
        </div>
      </div>

      <button class="btn btn-primary" on:click={backToSets}>
        Try Another Set
      </button>
    </div>
  {:else if questions.length > 0}
    <div class="quiz-header">
      <div class="quiz-nav">
        <button class="btn btn-secondary back-btn" on:click={backToSets}>
          <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
            <path d="M15.41 16.59L10.83 12l4.58-4.59L14 6l-6 6 6 6z"/>
          </svg>
          Back to Sets
        </button>
        
        <div class="quiz-progress">
          <span class="progress-text">Question {currentQuestionIndex + 1} of {questions.length}</span>
          <div class="progress-bar">
            <div 
              class="progress-fill" 
              style="width: {((currentQuestionIndex + 1) / questions.length) * 100}%"
            ></div>
          </div>
        </div>
      </div>

      <div class="timer {isTimeRunningOut ? 'warning' : ''} {isCriticalTime ? 'critical' : ''}">
        <div class="timer-icon">⏰</div>
        <div class="timer-display">
          <span class="time">{formatTime(minutes)}:{formatTime(seconds)}</span>
          <span class="timer-label">remaining</span>
        </div>
      </div>
    </div>

    <div class="question-card card">
      <div class="question-header">
        <h2 class="question-text">{questions[currentQuestionIndex].question}</h2>
        <div class="question-meta">
          <span class="chapter-badge">Chapter {questions[currentQuestionIndex].chapter}</span>
          <span class="marks-badge">{questions[currentQuestionIndex].marks} {questions[currentQuestionIndex].marks === 1 ? 'mark' : 'marks'}</span>
        </div>
      </div>
      
      <div class="options-container">
        {#each questions[currentQuestionIndex].options as option, i}
          <button
            class="option-btn {selectedAnswer === String.fromCharCode(97 + i) ? 'selected' : ''}"
            on:click={() => selectAnswer(String.fromCharCode(97 + i))}
            disabled={showExplanation}
          >
            <span class="option-letter">{String.fromCharCode(65 + i)}</span>
            <span class="option-text">{option}</span>
          </button>
        {/each}
      </div>

      <div class="question-actions">
        <div class="nav-buttons">
          {#if currentQuestionIndex > 0}
            <button class="btn btn-secondary" on:click={goToPreviousQuestion}>
              <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
                <path d="M15.41 16.59L10.83 12l4.58-4.59L14 6l-6 6 6 6z"/>
              </svg>
              Previous
            </button>
          {/if}
        </div>

        <button 
          class="btn btn-primary submit-btn" 
          on:click={submitAnswer} 
          disabled={!selectedAnswer || showExplanation}
        >
          Submit Answer
        </button>

        <div class="nav-buttons">
          {#if showExplanation}
            <button class="btn btn-success" on:click={goToNextQuestion}>
              {currentQuestionIndex < questions.length - 1 ? 'Next' : 'Finish'}
              <svg width="16" height="16" viewBox="0 0 24 24" fill="currentColor">
                <path d="M8.59 16.59L13.17 12L8.59 7.41L10 6l6 6-6 6z"/>
              </svg>
            </button>
          {/if}
        </div>
      </div>

      {#if showExplanation}
        <div class="explanation-card">
          <div class="explanation-header">
            <span class="explanation-icon">💡</span>
            <h3>Explanation</h3>
          </div>
          <div class="explanation-content">
            {@html questions[currentQuestionIndex].explanation}
          </div>
        </div>
      {/if}
    </div>
  {:else}
    <div class="loading-container">
      <div class="loading-spinner"></div>
      <p>Loading questions...</p>
    </div>
  {/if}
</div>

<style>
  .quiz-container {
    width: 100%;
    max-width: 800px;
    margin: 0 auto;
    padding: 1rem;
  }

  .quiz-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 2rem;
    gap: 1rem;
    flex-wrap: wrap;
  }

  .quiz-nav {
    display: flex;
    align-items: center;
    gap: 1rem;
    flex: 1;
  }

  .back-btn {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .quiz-progress {
    flex: 1;
    max-width: 300px;
  }

  .progress-text {
    font-size: 0.875rem;
    color: var(--text-secondary);
    margin-bottom: 0.5rem;
    display: block;
  }

  .progress-bar {
    width: 100%;
    height: 8px;
    background: var(--bg-tertiary);
    border-radius: 4px;
    overflow: hidden;
  }

  .progress-fill {
    height: 100%;
    background: linear-gradient(90deg, var(--primary-color), #8b5cf6);
    transition: width 0.3s ease-in-out;
  }

  .timer {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    padding: 1rem 1.5rem;
    background: var(--bg-primary);
    border: 2px solid var(--border-color);
    border-radius: var(--radius-lg);
    transition: all 0.3s ease-in-out;
  }

  .timer.warning {
    border-color: var(--warning-color);
    background: #fef3c7;
  }

  .timer.critical {
    border-color: var(--error-color);
    background: #fee2e2;
    animation: pulse 1s infinite;
  }

  .timer-icon {
    font-size: 1.25rem;
  }

  .timer-display {
    display: flex;
    flex-direction: column;
    align-items: center;
  }

  .time {
    font-size: 1.25rem;
    font-weight: 600;
    color: var(--text-primary);
    font-family: 'Courier New', monospace;
  }

  .timer-label {
    font-size: 0.75rem;
    color: var(--text-secondary);
  }

  .question-card {
    padding: 2.5rem;
  }

  .question-header {
    margin-bottom: 2rem;
  }

  .question-text {
    font-size: 1.25rem;
    font-weight: 600;
    color: var(--text-primary);
    line-height: 1.6;
    margin-bottom: 1rem;
  }

  .question-meta {
    display: flex;
    gap: 1rem;
    flex-wrap: wrap;
  }

  .chapter-badge,
  .marks-badge {
    padding: 0.25rem 0.75rem;
    border-radius: var(--radius-sm);
    font-size: 0.75rem;
    font-weight: 500;
  }

  .chapter-badge {
    background: var(--primary-light);
    color: var(--primary-color);
  }

  .marks-badge {
    background: var(--bg-tertiary);
    color: var(--text-secondary);
  }

  .options-container {
    display: flex;
    flex-direction: column;
    gap: 1rem;
    margin-bottom: 2rem;
  }

  .option-btn {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1.25rem;
    border: 2px solid var(--border-color);
    border-radius: var(--radius-lg);
    background: var(--bg-primary);
    cursor: pointer;
    transition: all 0.2s ease-in-out;
    text-align: left;
    width: 100%;
  }

  .option-btn:hover:not(:disabled) {
    border-color: var(--primary-color);
    background: var(--primary-light);
    transform: translateX(4px);
  }

  .option-btn.selected {
    border-color: var(--primary-color);
    background: var(--primary-color);
    color: white;
  }

  .option-btn:disabled {
    opacity: 0.7;
    cursor: not-allowed;
  }

  .option-letter {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 32px;
    height: 32px;
    border-radius: 50%;
    background: var(--bg-tertiary);
    font-weight: 600;
    font-size: 0.875rem;
    flex-shrink: 0;
  }

  .option-btn.selected .option-letter {
    background: rgba(255, 255, 255, 0.2);
    color: white;
  }

  .option-text {
    flex: 1;
    font-size: 1rem;
    line-height: 1.5;
  }

  .question-actions {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
  }

  .nav-buttons {
    min-width: 100px;
  }

  .submit-btn {
    flex: 1;
    max-width: 200px;
  }

  .explanation-card {
    margin-top: 2rem;
    padding: 1.5rem;
    background: var(--bg-secondary);
    border-radius: var(--radius-lg);
    border: 1px solid var(--border-color);
  }

  .explanation-header {
    display: flex;
    align-items: center;
    gap: 0.75rem;
    margin-bottom: 1rem;
  }

  .explanation-icon {
    font-size: 1.25rem;
  }

  .explanation-header h3 {
    font-size: 1.125rem;
    font-weight: 600;
    color: var(--text-primary);
  }

  .explanation-content {
    color: var(--text-secondary);
    line-height: 1.6;
  }

  .quiz-results {
    padding: 3rem;
    text-align: center;
    max-width: 500px;
    margin: 0 auto;
  }

  .results-header {
    margin-bottom: 2rem;
  }

  .results-icon {
    font-size: 3rem;
    margin-bottom: 1rem;
  }

  .results-header h2 {
    font-size: 2rem;
    font-weight: 700;
    color: var(--text-primary);
    margin-bottom: 0.5rem;
  }

  .results-subtitle {
    color: var(--text-secondary);
    font-size: 1rem;
  }

  .score-display {
    margin-bottom: 2rem;
  }

  .score-circle {
    display: inline-flex;
    align-items: baseline;
    justify-content: center;
    width: 120px;
    height: 120px;
    border-radius: 50%;
    background: linear-gradient(135deg, var(--primary-color), #8b5cf6);
    color: white;
    margin-bottom: 1rem;
    position: relative;
  }

  .score-number {
    font-size: 2.5rem;
    font-weight: 700;
  }

  .score-total {
    font-size: 1.25rem;
    font-weight: 500;
    opacity: 0.8;
  }

  .score-label {
    font-size: 1rem;
    color: var(--text-secondary);
  }

  .quiz-stats {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;
    margin-bottom: 2rem;
  }

  .stat {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 1rem;
    background: var(--bg-secondary);
    border-radius: var(--radius-lg);
  }

  .stat-value {
    font-size: 1.5rem;
    font-weight: 700;
    color: var(--primary-color);
  }

  .stat-label {
    font-size: 0.875rem;
    color: var(--text-secondary);
    margin-top: 0.25rem;
  }

  .loading-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 4rem 2rem;
    text-align: center;
  }

  .loading-spinner {
    width: 40px;
    height: 40px;
    border: 4px solid var(--border-color);
    border-top: 4px solid var(--primary-color);
    border-radius: 50%;
    animation: spin 1s linear infinite;
    margin-bottom: 1rem;
  }

  @keyframes spin {
    0% { transform: rotate(0deg); }
    100% { transform: rotate(360deg); }
  }

  @media (max-width: 768px) {
    .quiz-container {
      padding: 0.5rem;
    }

    .quiz-header {
      flex-direction: column;
      align-items: stretch;
      gap: 1rem;
    }

    .quiz-nav {
      flex-direction: column;
      align-items: stretch;
      gap: 1rem;
    }

    .question-card {
      padding: 1.5rem;
    }

    .question-text {
      font-size: 1.125rem;
    }

    .question-actions {
      flex-direction: column;
      gap: 1rem;
    }

    .nav-buttons {
      min-width: auto;
      width: 100%;
    }

    .submit-btn {
      max-width: none;
      order: -1;
    }

    .quiz-stats {
      grid-template-columns: 1fr;
      gap: 0.75rem;
    }

    .stat {
      padding: 0.75rem;
    }
  }
</style>