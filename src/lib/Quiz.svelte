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
  $: correctAnswer = questions[currentQuestionIndex]?.answer;
  $: isCorrect = selectedAnswer === correctAnswer;

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

  function getOptionClass(optionLetter) {
    if (!showExplanation) {
      // Before submission - show blue for selected option
      return selectedAnswer === optionLetter ? 'selected' : '';
    }
    
    // After submission - ALWAYS show green for correct answer
    if (optionLetter === correctAnswer) {
      return 'correct';
    } 
    // Show red only for the incorrect option that was selected
    else if (selectedAnswer === optionLetter && optionLetter !== correctAnswer) {
      return 'incorrect';
    }
    return '';
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
        <div class="question-number">Question {currentQuestionIndex + 1}</div>
        <h2 class="question-text">{questions[currentQuestionIndex].question}</h2>
        <div class="question-meta">
          <span class="chapter-badge">Chapter {questions[currentQuestionIndex].chapter}</span>
          <span class="marks-badge">{questions[currentQuestionIndex].marks} {questions[currentQuestionIndex].marks === 1 ? 'mark' : 'marks'}</span>
        </div>
      </div>
      
      <div class="options-container">
        <h3 class="options-title">Choose your answer:</h3>
        {#each questions[currentQuestionIndex].options as option, i}
          <button
            class="option-btn {getOptionClass(String.fromCharCode(97 + i))}"
            on:click={() => selectAnswer(String.fromCharCode(97 + i))}
            disabled={showExplanation}
          >
            <div class="option-letter">{String.fromCharCode(65 + i)}</div>
            <div class="option-content">
              <span class="option-text">{option}</span>
            </div>
            <div class="option-indicator">
              {#if showExplanation && String.fromCharCode(97 + i) === correctAnswer}
                <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z"/>
                </svg>
              {:else if showExplanation && selectedAnswer === String.fromCharCode(97 + i) && String.fromCharCode(97 + i) !== correctAnswer}
                <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"/>
                </svg>
              {:else if selectedAnswer === String.fromCharCode(97 + i) && !showExplanation}
                <svg width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M9 16.17L4.83 12l-1.42 1.41L9 19 21 7l-1.41-1.41z"/>
                </svg>
              {/if}
            </div>
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
          {showExplanation ? 'Answer Submitted' : 'Submit Answer'}
        </button>

        <div class="nav-buttons">
          {#if showExplanation}
            <button class="btn btn-success" on:click={goToNextQuestion}>
              {currentQuestionIndex < questions.length - 1 ? 'Next Question' : 'Finish Quiz'}
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
    color: var(--text-primary);
    margin-bottom: 0.5rem;
    display: block;
    font-weight: 600;
  }

  .progress-bar {
    width: 100%;
    height: 8px;
    background: var(--bg-tertiary);
    border-radius: 4px;
    overflow: hidden;
    border: 1px solid var(--border-color);
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
    color: #92400e;
  }

  .timer.critical {
    border-color: var(--error-color);
    background: #fee2e2;
    color: #991b1b;
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
    font-weight: 700;
    font-family: 'Courier New', monospace;
  }

  .timer-label {
    font-size: 0.75rem;
    font-weight: 500;
    opacity: 0.8;
  }

  .question-card {
    padding: 2rem;
    background: var(--bg-primary);
    border: 1px solid var(--border-color);
  }

  .question-header {
    margin-bottom: 2rem;
    border-bottom: 2px solid var(--border-color);
    padding-bottom: 1.5rem;
  }

  .question-number {
    font-size: 0.875rem;
    font-weight: 700;
    color: var(--primary-color);
    text-transform: uppercase;
    letter-spacing: 0.5px;
    margin-bottom: 0.75rem;
  }

  .question-text {
    font-size: 1.25rem;
    font-weight: 600;
    color: var(--text-primary);
    line-height: 1.6;
    margin-bottom: 1.25rem;
    text-align: left;
  }

  .question-meta {
    display: flex;
    gap: 1rem;
    flex-wrap: wrap;
  }

  .chapter-badge,
  .marks-badge {
    padding: 0.375rem 0.875rem;
    border-radius: var(--radius-sm);
    font-size: 0.75rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.5px;
  }

  .chapter-badge {
    background: var(--primary-color);
    color: white;
  }

  .marks-badge {
    background: var(--bg-tertiary);
    color: var(--text-primary);
    border: 1px solid var(--border-color);
  }

  .options-container {
    margin-bottom: 2rem;
  }

  .options-title {
    font-size: 1rem;
    font-weight: 600;
    color: var(--text-primary);
    margin-bottom: 1.5rem;
    text-align: left;
  }

  .option-btn {
    display: flex;
    align-items: center;
    gap: 1rem;
    padding: 1rem;
    margin-bottom: 0.75rem;
    border: 2px solid var(--border-color);
    border-radius: var(--radius-md);
    background: var(--bg-primary);
    cursor: pointer;
    transition: all 0.2s ease-in-out;
    text-align: left;
    width: 100%;
    position: relative;
    color: var(--text-primary);
  }

  /* Hover state - Blue background */
  .option-btn:hover:not(:disabled) {
    border-color: var(--option-hover-border);
    background: var(--option-hover-bg);
    color: var(--option-hover-color);
    transform: translateX(4px);
    box-shadow: var(--shadow-md);
  }

  /* Selected state (before submission) - Blue background */
  .option-btn.selected {
    border-color: var(--option-hover-border);
    background: var(--option-hover-bg);
    color: var(--option-hover-color);
    box-shadow: var(--shadow-lg);
  }

  /* Correct answer (after submission) - GREEN BACKGROUND ALWAYS */
  .option-btn.correct {
    border-color: var(--correct-border) !important;
    background: var(--correct-bg) !important;
    color: var(--correct-color) !important;
    box-shadow: 0 4px 12px rgba(22, 163, 74, 0.3) !important;
  }

  /* Incorrect answer (after submission) - Red background */
  .option-btn.incorrect {
    border-color: var(--incorrect-border);
    background: var(--incorrect-bg);
    color: var(--incorrect-color);
    box-shadow: 0 4px 12px rgba(220, 38, 38, 0.3);
  }

  .option-btn:disabled {
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
    font-weight: 700;
    font-size: 0.875rem;
    flex-shrink: 0;
    color: var(--text-primary);
    border: 2px solid var(--border-color);
    transition: all 0.2s ease-in-out;
  }

  /* Option letter styling for hover, selected, correct, and incorrect states */
  .option-btn:hover:not(:disabled) .option-letter,
  .option-btn.selected .option-letter,
  .option-btn.correct .option-letter,
  .option-btn.incorrect .option-letter {
    background: rgba(255, 255, 255, 0.2);
    color: white;
    border-color: rgba(255, 255, 255, 0.3);
  }

  .option-content {
    flex: 1;
    min-width: 0;
  }

  .option-text {
    font-size: 1rem;
    line-height: 1.5;
    font-weight: 500;
    display: block;
    word-wrap: break-word;
  }

  .option-indicator {
    width: 20px;
    height: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }

  .question-actions {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 1rem;
    padding-top: 1.5rem;
    border-top: 2px solid var(--border-color);
  }

  .nav-buttons {
    min-width: 120px;
  }

  .submit-btn {
    flex: 1;
    max-width: 200px;
    font-weight: 600;
  }

  .explanation-card {
    margin-top: 2rem;
    padding: 1.5rem;
    background: var(--bg-secondary);
    border-radius: var(--radius-lg);
    border: 2px solid var(--border-color);
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
    color: var(--text-primary);
    line-height: 1.6;
    font-size: 0.95rem;
    font-weight: 400;
  }

  .explanation-content :global(strong) {
    color: var(--primary-color);
    font-weight: 600;
  }

  .quiz-results {
    padding: 2.5rem;
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
    font-size: 1.75rem;
    font-weight: 700;
    color: var(--text-primary);
    margin-bottom: 0.5rem;
  }

  .results-subtitle {
    color: var(--text-secondary);
    font-size: 1rem;
    font-weight: 400;
  }

  .score-display {
    margin-bottom: 2rem;
  }

  .score-circle {
    display: inline-flex;
    align-items: baseline;
    justify-content: center;
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background: linear-gradient(135deg, var(--primary-color), #8b5cf6);
    color: white;
    margin-bottom: 1rem;
    position: relative;
  }

  .score-number {
    font-size: 2rem;
    font-weight: 700;
  }

  .score-total {
    font-size: 1rem;
    font-weight: 500;
    opacity: 0.8;
  }

  .score-label {
    font-size: 0.875rem;
    color: var(--text-primary);
    font-weight: 600;
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
    border: 1px solid var(--border-color);
  }

  .stat-value {
    font-size: 1.25rem;
    font-weight: 700;
    color: var(--primary-color);
  }

  .stat-label {
    font-size: 0.75rem;
    color: var(--text-primary);
    margin-top: 0.25rem;
    font-weight: 500;
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

  .loading-container p {
    color: var(--text-primary);
    font-weight: 500;
    font-size: 1rem;
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

    .option-btn {
      padding: 0.875rem;
      gap: 0.75rem;
    }

    .option-text {
      font-size: 0.9rem;
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