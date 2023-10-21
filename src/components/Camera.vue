<template>
  <div>
    <video ref="video" autoplay playsinline webkit-playsinline muted hidden></video>
    <canvas ref="canvas" :width="currentCameraMode.width" :height="currentCameraMode.height" class="bg-black rounded-3xl" :key="currentCameraMode.mode"></canvas>
    <div class="flex gap-40">
      <div>
        <input type="radio" id="cameraMode1" name="camera_mode" value="640x480" checked v-model="inputMode" @change="handleCameraModeChange"/>
        <label class="cursor-pointer" for="cameraMode1">640x480</label>

        <input type="radio" id="cameraMode2" name="contact" value="1280x720" v-model="inputMode" @change="handleCameraModeChange"/>
        <label class="cursor-pointer" for="cameraMode2">1280x720</label>
      </div>
    </div>
    <div class="flex gap-10 items-center justify-center py-4">
      <button class="px-6 py-4 bg-green-500 rounded
        text-white text-2xl uppercase font-bold
        hover:bg-green-600"
        @click="StartCamera">
        Start camera
      </button>
      <button class="px-6 py-4 bg-blue-500 rounded
        text-white text-2xl uppercase font-bold
        hover:bg-blue-600"
        @click="TakePicture">
        Take picture
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
const canvas = ref(null);
const video = ref(null);
const created = ref(false);
const cameraModesArray = [
  {
  mode: "640x480",
  width: 640,
  height: 480
  },
  {
  mode: "1280x720",
  width: 1280,
  height: 720
  }
];

const currentCameraMode = ref({
  mode: cameraModesArray[0].mode,
  width: cameraModesArray[0].width,
  height: cameraModesArray[0].height
})

const ctx = ref(null);
const inputMode = ref("");
export interface Constraints {
  audio: boolean,
  video: any
}
const constraints = ref<Constraints>({audio: false, video: true});
const SetStream = (stream) => {
  video.value.srcObject = stream;
  video.value.play();
  requestAnimationFrame(Draw)
}
const Draw = () => {
  ctx.value.drawImage(video.value, 0,0, canvas.value.width, canvas.value.height);
  requestAnimationFrame(Draw)
}
const TakePicture = () => {
  const link = document.createElement("a");
  link.download = `vue-cam-${new Date().toISOString()}.jpg`;
  link.href = canvas.value.toDataURL("image/jpeg");
  link.click();
}
const StartCamera = (async () => {
  created.value = true;
  if (video.value && canvas.value && created) {
    ctx.value = canvas.value.getContext("2d");
     await navigator.mediaDevices.getUserMedia(constraints.value)
         .then(SetStream)
         .catch(e => {console.error(e)})
  }
})
const handleCameraModeChange = () => {
  console.log(inputMode.value)

  const currentMode = cameraModesArray.find(e => e.mode === inputMode.value)
  console.log("currentMode", currentMode)
  if (currentMode) {
    currentCameraMode.value.mode = currentMode.mode;
    currentCameraMode.value.height = currentMode.height;
    currentCameraMode.value.width = currentMode.width;
    // class="cursor-pointer"
    constraints.value.video = {};
    constraints.value.video.width = currentMode.width;
    constraints.value.video.height = currentMode.height;
    // ctx.value = canvas.value.getContext("2d");
    // ctx.value.drawImage(video.value, 0,0, canvas.value.width, canvas.value.height);
  }
}
onMounted(async () => {
  canvas.value.width=currentCameraMode.value.width;
  canvas.value.height=currentCameraMode.value.height;
})
</script>

<style scoped>

</style>