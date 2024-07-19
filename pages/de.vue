<template>
  <div>
    <v-btn @click="requestCameraPermission">Request Camera Permission</v-btn>
    <v-btn @click="switchCamera('environment')">Back Camera</v-btn>
    <v-btn @click="switchCamera('user')">Front Camera</v-btn>
    <v-btn @click="takePhoto">Take Photo</v-btn>
    <div v-if="photo">
      <img :src="photo" alt="Captured Photo" />
    </div>
    <video ref="video" width="320" height="240" autoplay></video>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue';

const video = ref<HTMLVideoElement | null>(null);
const photo = ref<string | null>(null);
const currentStream = ref<MediaStream | null>(null);

const requestCameraPermission = async () => {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
    video.value!.srcObject = stream;
    currentStream.value = stream;
  } catch (err) {
    console.error('Error accessing camera:', err);
  }
};

const switchCamera = async (facingMode: 'user' | 'environment') => {
  if (currentStream.value) {
    currentStream.value.getTracks().forEach(track => track.stop());
  }

  try {
    const devices = await navigator.mediaDevices.enumerateDevices();
    const videoDevices = devices.filter(device => device.kind === 'videoinput');
    const device = videoDevices.find(device => device.label.toLowerCase().includes(facingMode));

    if (!device) {
      alert(`No ${facingMode === 'user' ? 'front' : 'back'} camera found on this device.`);
      return;
    }

    const stream = await navigator.mediaDevices.getUserMedia({
      video: { deviceId: device.deviceId }
    });
    video.value!.srcObject = stream;
    currentStream.value = stream;
  } catch (err) {
    console.error('Error switching camera:', err);
  }
};

const takePhoto = () => {
  if (!video.value) return;

  const canvas = document.createElement('canvas');
  canvas.width = video.value.videoWidth;
  canvas.height = video.value.videoHeight;
  canvas.getContext('2d')!.drawImage(video.value, 0, 0);

  photo.value = canvas.toDataURL('image/png');
};
</script>

<style scoped>
video {
  display: block;
  margin-bottom: 10px;
}
</style>
