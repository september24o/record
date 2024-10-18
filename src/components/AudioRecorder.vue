<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';
import axios from 'axios';

const isRecording = ref(false);
const audioChunks = ref<Blob[]>([]);
const mediaRecorder = ref<MediaRecorder | null>(null);
const audioUrl = ref('');
const responseText = ref('');
const INFERENCE_API = "http://url/v1/audio/transcriptions";
const modelUid = 'ChatTTS';

const startRecording = async () => {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    mediaRecorder.value = new MediaRecorder(stream);
    mediaRecorder.value.ondataavailable = (event) => {
      audioChunks.value.push(event.data);
    };
    mediaRecorder.value.onstop = () => {
      const audioBlob = new Blob(audioChunks.value, { type: 'audio/wav' });
      audioUrl.value = URL.createObjectURL(audioBlob);
    };
    mediaRecorder.value.start();
    isRecording.value = true;
  } catch (error) {
    console.error('Error accessing microphone:', error);
  }
};

const stopRecording = () => {
  if (mediaRecorder.value) {
    mediaRecorder.value.stop();
    isRecording.value = false;
  }
};

const submitRecording = async () => {
  if (audioChunks.value.length === 0) return;

  const audioBlob = new Blob(audioChunks.value, { type: 'audio/wav' });
  const formData = new FormData();
  formData.append('model', modelUid);
  formData.append('file', audioBlob, 'recording.wav');

  try {
    const response = await axios.post(INFERENCE_API, formData, {
      headers: {
        'Accept': 'application/json',
        'Content-Type': 'multipart/form-data'
      },
    });
    responseText.value = response.data.text || 'No response from the API';
  } catch (error) {
    console.error('Error submitting audio to API:', error);
    if (axios.isAxiosError(error) && error.response) {
      responseText.value = `Error: ${error.response.status} - ${error.response.data.message || 'Unknown error'}`;
    } else {
      responseText.value = 'Error submitting audio to API';
    }
  }
};

onMounted(() => {
  // Any setup code if needed
});

onUnmounted(() => {
  if (mediaRecorder.value && mediaRecorder.value.state !== 'inactive') {
    mediaRecorder.value.stop();
  }
});
</script>

<template>
  <div class="audio-recorder">
    <button @click="startRecording" :disabled="isRecording">Start Recording</button>
    <button @click="stopRecording" :disabled="!isRecording">Stop Recording</button>
    <button @click="submitRecording" :disabled="isRecording || audioChunks.length === 0">Submit Recording</button>
    
    <div v-if="audioUrl">
      <h3>Recorded Audio:</h3>
      <audio :src="audioUrl" controls></audio>
    </div>
    
    <div v-if="responseText">
      <h3>API Response:</h3>
      <p>{{ responseText }}</p>
    </div>
  </div>
</template>

<style scoped>
.audio-recorder {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

button {
  padding: 0.5rem 1rem;
  font-size: 1rem;
  cursor: pointer;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
