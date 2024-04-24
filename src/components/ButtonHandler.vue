<script setup>
import { ref, toRefs, defineProps } from "vue";
import { Webcam } from "../utils/webcam";

const props = defineProps({
  imageRef: {
    type: String,
  },
  cameraRef: {
    type: String,
  },
  videoRef: {
    type: String,
  },
});
const { imageRef, cameraRef, videoRef } = toRefs(props);

const streaming = ref(null); // streaming state
const inputImageRef = ref(null); // video input reference
const inputVideoRef = ref(null); // video input reference
const webcam = new Webcam(); // webcam handler

const closeImage = () => {
  const url = imageRef.value.src;
  imageRef.value.src = "#"; // restore image source
  URL.revokeObjectURL(url); // revoke url

  streaming.value = null;
  inputImageRef.value.value = "";
  imageRef.value.style.display = "none";
};

const closeVideo = () => {
  const url = videoRef.value.src;
  videoRef.value.src = ""; // restore video source
  URL.revokeObjectURL(url); // revoke url

  streaming.value = null;
  inputVideoRef.value.value = "";
  videoRef.value.style.display = "none";
};

const onImageUpload = (e) => {
  const url = URL.createObjectURL(e.target.files[0]); // create blob url
  imageRef.value.src = url;
  imageRef.value.style.display = "block";
  streaming.value = "image"; // set streaming to video
};

const onImageButtonClick = () => {
  // if not streaming
  if (streaming.value === null) {
    inputImageRef.value.click();
  } else if (streaming.value === "image") {
    closeImage();
  } else
    alert(
      `Can't handle more than 1 stream\nCurrently streaming : ${streaming.value}`
    );
};

const onVideoUpload = (e) => {
  if (streaming.value === "image") closeImage();

  const url = URL.createObjectURL(e.target.files[0]); // create blob url
  videoRef.value.src = url;
  videoRef.value.addEventListener("ended", () => closeVideo());
  videoRef.value.style.display = "block";
  streaming.value = "video";
};

const onWebcamButtonClick = () => {
  // if not streaming
  if (streaming.value === null || streaming.value === "image") {
    if (streaming.value === "image") closeImage();
    webcam.open(cameraRef.value);
    cameraRef.value.style.display = "block";
    streaming.value = "camera"; // set streaming to camera
  }
  // closing video streaming
  else if (streaming.value === "camera") {
    webcam.close(cameraRef.value);
    cameraRef.value.style.display = "none";
    streaming.value = null;
  } else
    alert(
      `Can't handle more than 1 stream\nCurrently streaming : ${streaming.value}`
    );
};

const onVideoButtonClick = () => {
  // if not streaming
  if (streaming.value === null || streaming.value === "image") {
    inputVideoRef.value.click();
  } else if (streaming.value === "video") {
    closeVideo();
  } else {
    alert(
      `Can't handle more than 1 stream\nCurrently streaming : ${streaming.value}`
    );
  }
};
</script>

<template>
  <div class="btn-container">
    <!-- Image Handler -->
    <input
      type="file"
      accept="image/*"
      :style="{ display: 'none' }"
      @change="onImageUpload"
      ref="inputImageRef"
    />
    <button @click="onImageButtonClick">
      {{ streaming === "image" ? "Close" : "Open" }} Image
    </button>

    <!-- Video Handler -->
    <input
      type="file"
      accept="video/*"
      :style="{ display: 'none' }"
      @change="onVideoUpload"
      ref="inputVideoRef"
    />
    <button @click="onVideoButtonClick">
      {{ streaming === "video" ? "Close" : "Open" }} Video
    </button>

    <!-- Webcam Handler -->
    <button @click="onWebcamButtonClick">
      {{ streaming === "camera" ? "Close" : "Open" }} Webcam
    </button>
  </div>
</template>

<style scoped>
button {
  text-decoration: none;
  color: white;
  background-color: black;
  border: 2px solid black;
  margin: 0 5px;
  padding: 5px;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  color: black;
  background-color: white;
  border: 2px solid black;
}
</style>
