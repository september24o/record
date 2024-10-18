<script setup lang="ts">
import AudioRecorder from './components/AudioRecorder.vue'
</script>

<template>
  <div class="app">
    <h1>Audio Recorder with CosyVoice API</h1>
    <AudioRecorder />
  </div>
</template>

<style scoped>
.app {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
}
</style>