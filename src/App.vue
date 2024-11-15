<template>
  <div>
    <div class="flex">
      <div class="video_container">
        <video ref="videoRef" id="video" width="640" height="480" autoplay></video>
        <canvas ref="canvasRef" width="640" height="480"></canvas>
      </div>
      <div class="picture_container">
        <canvas id="canvas" ref="canvas" width="640" height="480"></canvas>
      </div>
    </div>
    <button @click="start">Start</button>
    <button @click="stop">Stop</button>
    <button @click="pause">Pause</button>
  </div>
</template>
<script setup lang="ts">
import { ref, onMounted } from 'vue'
import * as faceApi from 'face-api.js'
const videoRef = ref()

const canvasRef = ref()

const canvas = ref()

onMounted(() => {
  loadModels()
})

//加载训练好的模型（weight，bias）
// ageGenderNet 识别性别和年龄
// faceExpressionNet 识别表情,开心，沮丧，普通
// faceLandmark68Net 识别脸部特征用于mobilenet算法
// faceLandmark68TinyNet 识别脸部特征用于tiny算法
// faceRecognitionNet 识别人脸
// ssdMobilenetv1 google开源AI算法除库包含分类和线性回归
// tinyFaceDetector 比Google的mobilenet更轻量级，速度更快一点
// mtcnn  多任务CNN算法，一开浏览器就卡死
// tinyYolov2 识别身体轮廓的算法，不知道怎么用
const loadModels = () => {
  try {
    Promise.all([
      faceApi.nets.tinyFaceDetector.loadFromUri('/weights'),
      faceApi.nets.faceLandmark68Net.loadFromUri('/weights'),
      faceApi.nets.ageGenderNet.loadFromUri('/weights')
      // faceApi.nets.faceRecognitionNet.loadFromUri('/weights'),
      // faceApi.nets.faceExpressionNet.loadFromUri('/weights')
    ]).then(() => {})
  } catch (error) {
    console.log(error)
  }
}

const start = async () => {
  try {
    let stream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true })
    videoRef.value.srcObject = stream
    const options = new faceApi.TinyFaceDetectorOptions({ inputSize: 416, scoreThreshold: 0.8 })
    setInterval(async () => {
      let detections = await faceApi.detectAllFaces(videoRef.value, options)
      // canvasRef.value.getContext('2d')?.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height)
      // faceApi.draw.drawDetections(canvasRef.value, detections, { withScore: true })
      const faceBoxes = detections.map((d) => d.box)
      drawImg(faceBoxes)
    }, 100)
  } catch (error) {
    console.log(error)
  }
}

//面部
const drawImg = (faceBoxes) => {
  const { x, y, width, height } = faceBoxes[0]
  canvas.value.getContext('2d')?.clearRect(0, 0, canvas.value.width, canvas.value.height)
  canvas.value.getContext('2d')?.drawImage(videoRef.value, x, y, width, height, 0, 0, canvas.value.width, canvas.value.height)
  // faceapi.draw.drawDetections(canvas.value, detections, { withScore: true })
  // const img = canvas.value.toDataURL('image/png')
  // const blob = base64ToBlob(img)
  // const imageUrl = URL.createObjectURL(blob)
  // console.log(imageUrl)
}
const base64ToBlob = (base64Data) => {
  var arr = base64Data.split(',')
  var mime = arr[0].match(/:(.*?);/)[1]
  var bstr = atob(arr[1])
  var n = bstr.length
  var u8arr = new Uint8Array(n)

  while (n--) {
    u8arr[n] = bstr.charCodeAt(n)
  }
  return new Blob([u8arr], { type: mime })
}
const stop = () => {}
const pause = () => {}
</script>
<style scoped lang="less">
.flex {
  display: flex;
}
.video_container {
  width: 640px;
  height: 480px;
  border: 1px solid #ccc;
  position: relative;
  * {
    position: absolute;
    top: 0;
    left: 0;
  }
}
.picture_container {
  width: 640px;
  height: 480px;
  border: 1px solid #ccc;
}
</style>
