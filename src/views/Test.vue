<template>
  <div class="ui container">
    <h2 class="ui header">Vocabulary Test</h2>
    <!-- Loading state -->
    <div v-if="loading" class="ui segment">
      <div class="ui active inline loader"></div> Loading...
    </div>
    <!-- If no vocabulary is found -->
    <div v-else-if="words.length === 0" class="ui segment">
      <h3>No vocabulary found. Please add some words first!</h3>
    </div>
    <!-- Quiz not started: choose how many words -->
    <div v-else-if="!quizStarted" class="ui segment" style="text-align:center">
      <h3>Choose number of words to test:</h3>
      <button class="ui blue button" @click="startQuiz(5)">Test 5 random words</button>
      <button class="ui green button" @click="startQuiz(10)" :disabled="words.length < 10">Test 10 random words</button>
      <p v-if="words.length < 10" style="margin-top:1em;color:#f2711c">
        <i class="exclamation icon"></i>Need at least 10 words to enable this option.
      </p>
    </div>
    <!-- Quiz finished: show score -->
    <div v-else-if="quizEnded" class="ui segment">
      <h3>Your final score: {{ score }} / {{ totalQuestions }}</h3>
      <button class="ui button primary" @click="resetQuiz">Choose test again</button>
    </div>
    <!-- Quiz in progress: show question and input -->
    <div v-else-if="currentWord" class="ui segment">
      <p><strong>Question {{ questionNumber }} of {{ totalQuestions }}</strong></p>
      <p><strong>Translate this word:</strong></p>
      <h3>{{ showVietnam ? currentWord.vietnam : currentWord.english }}</h3>
      <form class="ui form" @submit.prevent="checkAnswer">
        <div class="field">
          <input
            type="text"
            v-model="userAnswer"
            placeholder="Enter translation..."
            autocomplete="off"
            :disabled="answered"
          />
        </div>
        <button class="ui primary button" type="submit" :disabled="answered || userAnswer === ''">Check</button>
      </form>
      <!-- Feedback (correct or incorrect) -->
      <div v-if="feedback" :class="['ui message', isCorrect ? 'positive' : 'negative']" style="margin-top:10px">
        {{ feedback }}
      </div>
      <div class="ui divider"></div>
      <p><strong>Score:</strong> {{ score }}</p>
    </div>
  </div>
</template>

<script>
// Import API to fetch words
import { viewAllWords } from '@/helpers/api';

export default {
  name: 'Test',
  data() {
    return {
      words: [],             // All vocabulary
      quizWords: [],         // Words selected for the quiz
      currentWord: null,     // The word being asked
      showVietnam: true,     // true: show Vietnamese, false: English
      userAnswer: '',        // User input
      score: 0,              // Current score
      feedback: '',          // Feedback message
      isCorrect: false,      // Whether the last answer was correct
      questionNumber: 1,     // Current question number
      totalQuestions: 0,     // Total number of questions
      quizEnded: false,      // Quiz ended flag
      answered: false,       // Whether user has answered current question
      quizStarted: false,    // Quiz started flag
      loading: true,         // Loading state
    };
  },

  // Fetch all words when component mounts
  async mounted() {
    const res = await viewAllWords();
    this.words = Array.isArray(res) ? res : [];
    this.loading = false;
  },

  methods: {
    // Start the quiz, pick n random words
    startQuiz(num) {
      this.quizWords = this.shuffle(this.words).slice(0, num);
      this.totalQuestions = this.quizWords.length;
      this.questionNumber = 1;
      this.score = 0;
      this.quizEnded = false;
      this.quizStarted = true;
      this.nextWord();
    },

    // Shuffle the array
    shuffle(arr) {
      const a = arr.slice();
      for (let i = a.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [a[i], a[j]] = [a[j], a[i]];
      }
      return a;
    },

    // Move to next word/question
    nextWord() {
      if (this.questionNumber > this.totalQuestions) {
        this.quizEnded = true;
        return;
      }
      this.currentWord = this.quizWords[this.questionNumber - 1];
      this.userAnswer = '';
      this.feedback = '';
      this.isCorrect = false;
      this.answered = false;
      // Randomly decide which language to show
      this.showVietnam = Math.random() < 0.5;
    },

    // Check if the answer is correct
    checkAnswer() {
      if (!this.currentWord || this.answered) return;
      const correctAnswer = (this.showVietnam ? this.currentWord.english : this.currentWord.vietnam)
        .trim()
        .toLowerCase();
      const userInput = this.userAnswer.trim().toLowerCase();
      this.answered = true;
      if (userInput === correctAnswer) {
        this.score++;
        this.feedback = '✅ Correct!';
        this.isCorrect = true;
      } else {
        this.feedback = `❌ Incorrect. Correct answer: "${correctAnswer}"`;
        this.isCorrect = false;
      }
      // After 1.2s, go to next question or finish
      setTimeout(() => {
        this.questionNumber++;
        this.nextWord();
      }, 1200);
    },

    // Reset to initial state to choose again
    resetQuiz() {
      this.quizStarted = false;
      this.quizEnded = false;
      this.questionNumber = 1;
      this.userAnswer = '';
      this.currentWord = null;
      this.feedback = '';
      this.score = 0;
    }
  }
};
</script>

<style scoped>
/* Container for the test page */
.ui.container {
  max-width: 480px !important;
  min-height: 95vh;
  margin: 48px auto !important;
  background: #f7f8fd;
  border-radius: 18px;
  box-shadow: 0 6px 32px #c1c5df1a, 0 1.5px 2px #e4e6fb7a;
  padding: 28px 0 22px 0 !important;
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* Quiz title style: big, gradient */
.ui.header {
  font-size: 2.1rem;
  text-align: center;
  margin-bottom: 24px;
  font-weight: 700;
  letter-spacing: 1.5px;
  background: linear-gradient(90deg,#f7971e,#ffd200,#21d4fd,#b721ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

/* Card-like segment for each quiz step */
.ui.segment {
  background: #fff;
  border-radius: 14px !important;
  box-shadow: 0 2px 12px #ede9fc39;
  margin: 0 auto 26px auto !important;
  padding: 26px 28px !important;
  min-width: 280px;
}

/* Textbox input for answer */
.ui.form input[type="text"] {
  border-radius: 9px;
  border: 1.5px solid #d5d9fa;
  background: #fafbff;
  padding: 12px 14px;
  font-size: 1.1rem;
  margin-bottom: 8px;
  transition: border .16s;
}
.ui.form input[type="text"]:focus {
  border: 1.5px solid #a1b6ff;
  background: #fff;
  outline: none;
}

/* All buttons: colorful, rounded, bold */
.ui.primary.button,
.ui.blue.button,
.ui.green.button {
  font-size: 1.1rem;
  border-radius: 9px !important;
  font-weight: 600;
  padding: 10px 22px !important;
  margin: 8px 6px 0 0;
  background: linear-gradient(90deg,#ffd200,#21d4fd)!important;
  color: #2b2a41 !important;
  box-shadow: 0 2px 12px #ffe5a444;
  border: none;
  transition: box-shadow 0.14s;
}
.ui.primary.button:hover,
.ui.blue.button:hover,
.ui.green.button:hover {
  box-shadow: 0 4px 18px #ffd20044;
  color: #242541 !important;
}

/* Disabled button */
.ui.button[disabled] {
  opacity: 0.5 !important;
  cursor: not-allowed !important;
}

.ui.divider {
  margin: 18px 0 !important;
}

/* Feedback message (right or wrong) */
.ui.message {
  font-size: 1.1rem;
  padding: 10px 16px !important;
  border-radius: 8px !important;
  margin-bottom: 8px;
}
.ui.message.positive {
  background: #eaffea !important;
  color: #2e7d32 !important;
  border-color: #b3e6be !important;
}
.ui.message.negative {
  background: #fff0f0 !important;
  color: #c62828 !important;
  border-color: #f5bcbc !important;
}

/* Mobile responsive style */
@media (max-width: 600px) {
  .ui.container {
    padding: 4vw 2vw !important;
    min-width: unset;
    max-width: 99vw !important;
    box-shadow: none;
  }
  .ui.segment {
    padding: 18px 3vw !important;
  }
}
</style>
