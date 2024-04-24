<script setup>
import { ref, onMounted } from "vue";
import Loader from "./Loader.vue";
import ButtonHandler from "./ButtonHandler.vue";
import * as tf from "@tensorflow/tfjs";

import "@tensorflow/tfjs-backend-webgl"; //
import { detect, detectVideo } from "../utils/detect";

const loading = ref({ loading: true, progress: 0 });

let model = {
  net: null,
  inputShape: [1, 0, 0, 3],
}; // init model & input shape

// references
const imageRef = ref(null);
const cameraRef = ref(null);
const videoRef = ref(null);
const canvasRef = ref(null);

// model configs
const modelName = "yolov8n";

onMounted(() => {
  tf.ready().then(async () => {
    const yolov8 = await tf.loadGraphModel(
      `${window.location.href}/${modelName}_web_model/model.json`,
      {
        onProgress: (fractions) => {
          loading.value = { loading: true, progress: fractions }; // set loading fractions
        },
      }
    ); // load model

    // warming up model
    const dummyInput = tf.ones(yolov8.inputs[0].shape);
    const warmupResults = yolov8.execute(dummyInput);

    loading.value = { loading: false, progress: 1 };
    model = {
      net: yolov8,
      inputShape: yolov8.inputs[0].shape,
    }; // set model & input shape

    tf.dispose([warmupResults, dummyInput]); // cleanup memory
  });
});
</script>

<template>
  <div class="container">
    <Loader v-if="loading.loading"
      >Loading model... {{ (loading.progress * 100).toFixed(2) }}%</Loader
    >
    <template v-else>
      <div class="header">
        <h1>YOLOv8 Live Detection App</h1>
        <p>
          YOLOv8 live detection application on browser powered by
          <code>tensorflow.js</code>
        </p>
        <p>
          Serving : <code class="code">{{ modelName }}</code>
        </p>
      </div>

      <div class="content">
        <img
          src="#"
          ref="imageRef"
          @load="detect(imageRef, model, canvasRef)"
        />
        <video
          autoPlay
          muted
          ref="cameraRef"
          @play="detectVideo(cameraRef, model, canvasRef)"
        />
        <video
          autoPlay
          muted
          ref="videoRef"
          @play="detectVideo(videoRef, model, canvasRef)"
        />
        <canvas
          :width="model.inputShape[1]"
          :height="model.inputShape[2]"
          ref="canvasRef"
        />
      </div>

      <ButtonHandler
        :imageRef="imageRef"
        :cameraRef="cameraRef"
        :videoRef="videoRef"
      />
    </template>
  </div>
</template>

<style></style>
