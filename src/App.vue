<template>
  <div id="app">
    <button type="button" class="btn btn-start" @click="startMic" v-show="!streamStarted">Start</button>
    <button type="button" class="btn btn-stop" @click="stopMic" v-show="streamStarted">Stop</button>
    <Tuner :frequency="frequency" />
  </div>
</template>

<script>
import Tuner from "./components/Tuner.vue";

export default {
  name: 'App',

  components: {
    Tuner
  },

  data() {
    return {
      frequency: 440,
      stream: null,
      streamStarted: false,
      frequencyUpdateInterval: null,
    };
  },

  methods: {
    getLocalStream() {
      return navigator.mediaDevices.getUserMedia({ video: false, audio: true });
    },

    async startMic() {
      this.stream = await this.getLocalStream();
      this.streamStarted = true;
      this.updateFrequencies();
    },

    async stopMic() {
      this.stream.getTracks().forEach((track) => track.stop());
      this.streamStarted = false;

      clearInterval(this.frequencyUpdateInterval);
    },

    async updateFrequencies() {
      const audioContext = new AudioContext();

      const mediaStreamSource = audioContext.createMediaStreamSource(this.stream);
      const analyser = audioContext.createAnalyser();
      analyser.fftSize = 32768;

      mediaStreamSource.connect(analyser);

      const data = new Float32Array(analyser.frequencyBinCount);

      this.frequencyUpdateInterval = setInterval(() => {
        analyser.getFloatFrequencyData(data);

        let idx = 0;

        for (let j = 0; j < analyser.frequencyBinCount; j++) {
          if (data[j] > data[idx]) {
            idx = j;
          }
        }

        const frequency = idx * audioContext.sampleRate / analyser.fftSize;

        this.frequency = frequency;
      }, 200);
    },
  }
}
</script>

<style>
body {
  background-color: #e8e8e9;
}

.btn {
  display: flex;
  margin: 3em auto 0;
  color: #fff;
  border: 1px solid #329a4a;
  border-radius: 3px;
  padding: 5px 30px;
  font-family: sans-serif;
  font-size: 1.5rem;
}

.btn-start {
  background-color: #329a4a;
}

.btn-stop {
  background-color: #ec370a;
}

.btn-start:hover {
  background-color: #2b833f;
  border-color: #2b833f;
}

.btn-start:active {
  background-color: #37b757;
  border-color: #37b757;
}

.btn:disabled {
  background-color: #a9a9a9;
  border-color: #a9a9a9;
}
</style>
