<template>
  <div class="container">
    <div class="header">
      <h1>Typing Speed ​​Test</h1>
    </div>

    <div class="controls">
      <div class="timer">{{ formattedTime }}</div>
    </div>

    <div class="text-display">
      {{ currentText }}
    </div>

    <textarea
      v-model="userInput"
      placeholder="Start typing..."
      @input="handleInput"
    ></textarea>

    <div class="stats">
      <div class="stat">
        <div class="stat-label">Accuracy</div>
        <div class="stat-value">{{ accuracy }}%</div>
      </div>
      <div class="stat">
        <div class="stat-label">Words/Minute</div>
        <div class="stat-value">{{ wpm }}</div>
      </div>
    </div>

    <div class="result" :class="{ show: isFinished }">
      <div class="result-title">Test Results</div>
      <div class="result-detail">Total Time: {{ formattedTime }}</div>
      <div class="result-detail">
        Correct Chars: <span class="correct">{{ correctChars }}</span>
      </div>
      <div class="result-detail">
        Wrong Chars: <span class="error">{{ wrongChars }}</span>
      </div>
      <div class="result-detail">
        Final WPM: {{ finalWpm }} word/minute
      </div>
      <div class="result-detail">
        Final Accuracy: {{ finalAccuracy }}%
      </div>
    </div>

    <button
      class="restart-btn"
      :disabled="!isActive && !isFinished"
      @click="resetTest"
    >
      Restart
    </button>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted, onMounted } from "vue";

const userInput = ref("");
const text = ref("");
const isActive = ref(false);
const isFinished = ref(false);
const startTime = ref(null);
const elapsedTime = ref(0);
const timerInterval = ref(null);
const correctChars = ref(0);
const wrongChars = ref(0);
const finalWpm = ref(0);
const finalAccuracy = ref(0);

const currentText = computed(() => text.value);

const formattedTime = computed(() => {
  const minutes = Math.floor(elapsedTime.value / 60)
    .toString()
    .padStart(2, "0");
  const seconds = (elapsedTime.value % 60).toString().padStart(2, "0");
  return `${minutes}:${seconds}`;
});

const accuracy = computed(() => {
  if (!userInput.value.length) return 0;
  const correct = userInput.value
    .split("")
    .filter((char, i) => char === currentText.value[i]).length;
  return Math.round((correct / userInput.value.length) * 100);
});

const wpm = computed(() => {
  if (!elapsedTime.value) return 0;
  const minutes = elapsedTime.value / 60;
  const words = userInput.value.length / 5;
  return Math.round(words / minutes);
});

// Methods
function startTest() {
  isActive.value = true;
  startTime.value = Date.now();
  timerInterval.value = setInterval(() => {
    elapsedTime.value = Math.floor((Date.now() - startTime.value) / 1000);
  }, 1000);
}

function resetTest() {
  clearInterval(timerInterval.value);
  userInput.value = "";
  isActive.value = false;
  isFinished.value = false;
  elapsedTime.value = 0;
  correctChars.value = 0;
  wrongChars.value = 0;
  finalWpm.value = 0;
  finalAccuracy.value = 0;
  getRandomText();
}

function finishTest() {
  isFinished.value = true;
  isActive.value = false;
  clearInterval(timerInterval.value);

  const text = currentText.value;
  const input = userInput.value;
  let correct = 0;
  let wrong = 0;

  const maxLength = Math.max(text.length, input.length);
  for (let i = 0; i < maxLength; i++) {
    if (i < text.length && i < input.length) {
      if (input[i] === text[i]) {
        correct++;
      } else {
        wrong++;
      }
    } else {
      wrong++;
    }
  }

  correctChars.value = correct;
  wrongChars.value = wrong;
  finalWpm.value = wpm.value;
  finalAccuracy.value = Math.round((correct / (correct + wrong)) * 100);
}

function handleInput() {
  if (!isActive.value) {
    startTest();
  }
  if (userInput.value.length >= currentText.value.length) {
    finishTest();
  }
}

const getRandomText = async () => {
  try {
    const response = await fetch("https://api.quotable.io/random?minLength=150&maxLength=180");
    const data = await response.json();
    text.value = data.content;
  } catch (error) {
    console.error("Hata oluştu:", error);
  }
};

onUnmounted(() => {
  clearInterval(timerInterval.value);
});

onMounted(() => {
  getRandomText();
});
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
}

.container {
  background: rgba(255, 255, 255, 0.95);
  padding: 2rem;
  border-radius: 15px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 800px;
}

.header {
  text-align: center;
  margin-bottom: 2rem;
}

.header h1 {
  color: #2d3748;
  margin-bottom: 0.5rem;
  font-size: 22px;
}

.controls {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

select {
  padding: 0.5rem 1rem;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  background-color: white;
  color: #4a5568;
  font-size: 1rem;
  outline: none;
  transition: border-color 0.2s;
  width: 120px;
}

select:focus {
  border-color: #667eea;
}

.timer {
  padding: 0.5rem 1rem;
  background-color: #667eea;
  color: white;
  border-radius: 8px;
  font-weight: 600;
  min-width: 100px;
  text-align: center;
}

.text-display {
  background-color: #f7fafc;
  padding: 1rem;
  border-radius: 8px;
  margin-bottom: 1rem;
  min-height: 100px;
  font-size: 1.1rem;
  line-height: 1.6;
  color: #2d3748;
}

textarea {
  width: 100%;
  height: 150px;
  padding: 1rem;
  border: 2px solid #e2e8f0;
  border-radius: 8px;
  font-size: 1.1rem;
  line-height: 1.6;
  resize: none;
  outline: none;
  transition: border-color 0.2s;
  background-color: #ddd;
  color: #222;
}

textarea:focus {
  border-color: #667eea;
}

.stats {
  margin-top: 1rem;
  display: flex;
  justify-content: space-around;
  padding: 1rem;
  background-color: #f7fafc;
  border-radius: 8px;
}

.stat {
  text-align: center;
}

.stat-label {
  font-size: 0.875rem;
  color: #718096;
  margin-bottom: 0.25rem;
}

.stat-value {
  font-size: 1.25rem;
  font-weight: 600;
  color: #2d3748;
}

.result {
  margin-top: 1rem;
  padding: 1rem;
  background-color: #f7fafc;
  border-radius: 8px;
  display: none;
}

.result.show {
  display: block;
}

.result-title {
  font-size: 1.25rem;
  font-weight: 600;
  color: #2d3748;
  margin-bottom: 1rem;
  text-align: center;
}

.result-detail {
  margin: 0.5rem 0;
  color: #4a5568;
}

.restart-btn {
  display: block;
  width: 200px;
  margin: 1rem auto;
  padding: 0.75rem 1.5rem;
  background-color: #667eea;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s;
}

.restart-btn:hover {
  background-color: #5a67d8;
}

.restart-btn:disabled {
  background-color: #cbd5e0;
  cursor: not-allowed;
}

.error {
  color: #e53e3e;
}

.correct {
  color: #38a169;
}
</style>
