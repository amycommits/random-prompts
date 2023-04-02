<template>
  <div class="home">
    <input
      v-model="snippet"
      type="text"
      style="width: 100%; margin-bottom: 5px"
    />
    <textarea v-model="article" rows="50" style="width: 100%" />
    <button @click="handleClick" @keydown.enter="handleClick">Create</button>
    <div v-if="prompts">
      <div
        v-for="(prompt, index) in prompts"
        class="prompt-container"
        :key="index"
        :class="{
          'active-selected': index === selectedPrompt,
          selected: selectedPrompts.includes(index) && index !== selectedPrompt,
        }"
        @click="copyToClipboard(prompt, index)"
      >
        {{ prompt }}
      </div>
    </div>
  </div>
</template>

<script>
import ArtInfo from "@/utils/art_info.js";

export default {
  name: "HomeView",
  data() {
    return {
      article: "",
      articleFrequentWords: "",
      selectedPrompts: [],
      selectedPrompt: null,
      snippet: "",
      prompts: [],
    };
  },
  methods: {
    handleClick() {
      this.prompts = [];
      this.prompts.push(this.snippet);
      this.articlePrompt();
      Array.from({ length: 5 }, () => {
        const addons = [];
        addons.push(this.getRandomElements(ArtInfo.artMediums, 0, 5));
        addons.push(this.getRandomElements(ArtInfo.artStyles, 0, 5));
        addons.push(this.getRandomElements(ArtInfo.artists, 0, 5));
        this.prompts.push(`${this.articleFrequentWords}, ${addons.join(",")}`);
      });
      Array.from({ length: 5 }, () => {
        const addons = [];
        addons.push(this.getRandomElements(ArtInfo.artMediums, 0, 5));
        addons.push(this.getRandomElements(ArtInfo.artStyles, 0, 5));
        addons.push(this.getRandomElements(ArtInfo.artists, 0, 5));
        this.prompts.push(`${this.snippet}, ${addons.join(",")}`);
      });
    },
    articlePrompt() {
      // Count the frequency of words in the article
      const emptyWords = [
        "the",
        "to",
        "a",
        "an",
        "and",
        "or",
        "in",
        "of",
        "is",
        "for",
        "you",
        "can",
        "that",
        "on",
        "are",
        "how",
        "more",
        "your",
        "their",
        "this",
        "it",
        "with",
        "they",
        "at",
        "these",
        "be",
        "as",
      ];
      const wordFrequency = this.article
        .split(" ")
        .filter((word) => {
          return !emptyWords.includes(word);
        })
        .reduce((acc, word) => {
          word = word.toLowerCase();
          acc[word] = (acc[word] || 0) + 1;
          return acc;
        }, {});

      // Sort the words by frequency and take the top 10 most frequent words
      const topWords = Object.entries(wordFrequency)
        .sort((a, b) => b[1] - a[1])
        .slice(0, 10)
        .map(([word]) => word);

      // Join the top 10 words into a string
      const topWordsString = topWords.join(", ");

      // Add the top 10 words to the prompts array
      // this.prompts.push(topWordsString);
      this.articleFrequentWords = topWordsString;
      this.prompts.push(topWordsString);
      // return topWordsString;
    },
    getElement(el, min, max) {
      return this.getRandomElements(el, min, max);
    },

    getRandomElements(arr, min, max) {
      const randomCount = Math.floor(Math.random() * (max - min + 1) + min);
      const shuffled = arr.sort(() => 0.5 - Math.random());
      return shuffled.slice(0, randomCount);
    },
    copyToClipboard(prompt, index) {
      this.selectedPrompt = index;
      if (!this.selectedPrompts.includes(index)) {
        this.selectedPrompts.push(index);
      }
      const el = document.createElement("textarea");
      el.value = prompt;
      document.body.appendChild(el);
      el.select();
      document.execCommand("copy");
      document.body.removeChild(el);
    },
  },
};
</script>
<style scoped>
.prompt-container {
  margin: 0 5px;
  width: 100vw;
}
.active-selected {
  background-color: darkblue;
  color: white;
}
.selected {
  background-color: lightblue;
  color: black;
}
</style>
