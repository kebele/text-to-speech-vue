<template>
  <!-- 
************
  THANKS TO GOKHAN KANDEMIR (KABLOSUZ KEDI)
  https://www.youtube.com/watch?v=8drgGBKRClE
**************
 -->
  <div id="app">
    <appHeader />
    <div class="app-container">
      <!-- <p>{{ selectedVoice }}</p> -->
      <select v-model="selectedVoice">
        <option v-for="voice in voiceList" :key="voice.voiceURI">
          <!-- {{ `${voice.name}- ${voice.lang}` }} -->
          {{ voice.name }}
        </option>
      </select>
      <br />
      <div class="slidercontainer">
        <input
          v-model="speed"
          ref="speed"
          class="slider"
          type="range"
          min="0.5"
          max="3"
        />
        <!-- {{ speed }} -->
      </div>
      <span> </span>
      <br />
      <textarea
        name=""
        id="textToSpeech"
        cols="30"
        rows="10"
        v-model="textToSpeech"
      ></textarea>
      <div class="previewContainer">
        <span
          v-for="(word, i) in textToSpeech.split(' ')"
          :key="i"
          :ref="`spoken_word_${i}`"
        >
          {{ word }}
        </span>
      </div>
      <br />
      <button class="btn red" type="button" @click="speak">
        <span>Söyle Bakalım Şekerim!</span>
      </button>
    </div>
  </div>
</template>

<script>
import appHeader from "@/components/appHeader.vue";
export default {
  name: "App",
  components: {
    appHeader,
  },
  created() {
    // console.log(this.tts.getVoices());
    // console.log(this.getVoices());
    this.getVoices().then((voices) => {
      // console.log("voices", voices);
      this.voiceList = voices;
      // this.selectedVoice = this.voiceList.find((v) => v.name == "Yelda").name;
      this.selectedVoice = "Yelda";
    });
  },
  mounted() {
    this.$refs.speed.step = 0.5;
  },
  data() {
    return {
      tts: window.speechSynthesis,
      voiceList: null,
      selectedVoice: null,
      textToSpeech: "lorem ipsum dolor sit amet",
      speed: 1,
    };
  },
  methods: {
    getVoices() {
      let intervalID;
      return new Promise((resolve) => {
        intervalID = setInterval(() => {
          if (this.tts.getVoices().length > 0) {
            resolve(this.tts.getVoices());
            clearInterval(intervalID);
          }
        }, 10);
      });
    },
    speak() {
      let toSpeak = new SpeechSynthesisUtterance(this.textToSpeech);
      const foundActiveVoice =
        this.voiceList.find((v) => v.name == this.selectedVoice) || null;
      toSpeak.voice = foundActiveVoice;
      toSpeak.rate = this.speed;
      toSpeak.onboundary = this.onBoundary;
      this.tts.speak(toSpeak);
    },
    onBoundary(event) {
      // console.log(event);

      const spokenWord = event.utterance.text.substr(
        event.charIndex,
        event.charLength
      );
      const wordIndex = this.textToSpeech
        .split(" ")
        .findIndex((word) => word == spokenWord);

      this.$refs[`spoken_word_${wordIndex}`][0].classList.add("spoken_word");

      console.log("söylediği kelime ve indexi : ", spokenWord, wordIndex);
    },
  },
};
</script>
