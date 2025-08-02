<template>
  <!-- Main wrapper for the New Word page -->
  <div>
    <h1>New Word</h1>
    <form @submit.prevent="onSubmit">
      <!-- English input field -->
      <div class="ui labeled input fluid">
        <div class="ui label"><i class="united kingdom flag"></i> English</div>
        <input type="text" required v-model="word.english" />
      </div>
      <br />
      <!-- Vietnamese input field -->
      <div class="ui labeled input fluid">
        <div class="ui label"><i class="vietnam flag"></i> Vietnam</div>
        <input type="text" required v-model="word.vietnam" />
      </div>
      <br />
      <!-- Submit button -->
      <button class="ui primary button">Submit</button>
    </form>
  </div>
</template>

<script setup>
// Vue composition API & router imports
import { ref, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import { viewAllWords, addNewWord } from "../helpers/api";

// State for new word and the list of all words
const word = ref({ english: '', vietnam: '' });
const words = ref([]);
const router = useRouter();

// Fetch all existing words on page load (for duplicate check)
onMounted(async () => {
  words.value = (await viewAllWords()) || [];
});

// Submit handler: check duplicate, confirm, then add word
const onSubmit = async () => {
  // Check for duplicates (ignore case, both English and Vietnamese)
  const duplicates = words.value.filter(
    w =>
      w.english.trim().toLowerCase() === word.value.english.trim().toLowerCase() ||
      w.vietnam.trim().toLowerCase() === word.value.vietnam.trim().toLowerCase()
  );
  if (duplicates.length > 0) {
    const dupString = duplicates
      .map(w => `"${w.english}" - "${w.vietnam}"`)
      .join(', ');
    const confirmAdd = window.confirm(
      `This word was added before: ${dupString}\nDo you want to add more?`
    );
    if (!confirmAdd) return;
  }
  await addNewWord(word.value);
  router.push('/words');
};
</script>

<style scoped>
/* Page background and vertical center */
body, .newword-bg {
  min-height: 96vh;
  background: #f6f7fb;
}
.newword-bg {
  display: flex;
  align-items: center;
  justify-content: center;
  min-height: 96vh;
}

/* Card-like wrapper for the form */
.newword-card {
  background: #fff;
  border-radius: 20px;
  box-shadow: 0 6px 32px #c1c5df1a, 0 1.5px 2px #e4e6fb7a;
  padding: 38px 34px 30px 34px;
  max-width: 400px;
  width: 100%;
  margin: 44px auto 0 auto;
  display: flex;
  flex-direction: column;
}

/* Title: big, centered, gradient color */
h1 {
  font-size: 2.1rem;
  text-align: center;
  margin-bottom: 26px;
  font-weight: 700;
  letter-spacing: 1.5px;
  background: linear-gradient(90deg,#f7971e,#ffd200,#21d4fd,#b721ff);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

/* Form layout: column, with spacing */
form {
  display: flex;
  flex-direction: column;
  gap: 22px;
}

/* Input group styling: flex, soft border, highlight on focus */
.ui.labeled.input.fluid {
  display: flex;
  align-items: center;
  border-radius: 9px;
  border: 1.5px solid #e1e5fc;
  background: #f8f9ff;
  transition: border .15s;
  padding: 0 6px 0 0;
}
.ui.labeled.input.fluid:focus-within {
  border: 1.5px solid #a1b6ff;
  background: #fff;
}

/* Label style: color, rounded left, icon inline */
.ui.labeled.input.fluid .ui.label {
  background: #f7d94c2a;
  color: #6d6c8b;
  font-weight: 600;
  border-radius: 9px 0 0 9px;
  font-size: 1.07rem;
  min-width: 105px;
  padding-left: 10px;
  letter-spacing: .5px;
  border: none;
  display: flex;
  align-items: center;
  gap: 7px;
}

/* Text input: clean, padded, rounded right */
.ui.labeled.input.fluid input[type="text"] {
  border: none;
  background: transparent;
  outline: none;
  padding: 11px 12px;
  font-size: 1.07rem;
  border-radius: 0 9px 9px 0;
  flex: 1;
  color: #333;
}

/* Submit button: gradient, rounded, full width, hover effect */
button.ui.primary.button {
  width: 100%;
  font-size: 1.08rem;
  border-radius: 10px;
  font-weight: 600;
  margin-top: 8px;
  background: linear-gradient(90deg,#ffd200,#21d4fd)!important;
  color: #2b2a41 !important;
  box-shadow: 0 2px 12px #ffe5a444;
  border: none;
  transition: box-shadow 0.14s;
}
button.ui.primary.button:hover {
  box-shadow: 0 4px 20px #ffd20044;
}

/* Responsive: padding and label size */
@media (max-width: 550px) {
  .newword-card {
    padding: 18px 6vw 18px 6vw;
  }
  .ui.labeled.input.fluid .ui.label {
    font-size: 0.98rem;
    min-width: 70px;
    padding-left: 4px;
  }
}
</style>
