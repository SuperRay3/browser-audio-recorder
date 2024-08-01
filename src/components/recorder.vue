<script setup>
import { ref } from "vue";

const isRecording = ref(false)
const audioPlayerRef = ref(null)

let mediaRecorder // 录音器实例
let audioChunks = [] // 录制的音频片段
let audioBlob // 录音的 blob 文件

async function startRecording() {
  audioChunks = []
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true })
    mediaRecorder = new MediaRecorder(stream)
    mediaRecorder.ondataavailable = event => {
      audioChunks.push(event.data)
    }

    mediaRecorder.start()
    isRecording.value = true
  } catch(error) {
    console.error(error)
  }
}

function finishRecording() {
  mediaRecorder.stop()
  
  mediaRecorder.onstop = () => {
    audioBlob = new Blob(audioChunks, { type: 'audio/wav' })
    isRecording.value = false
  }
}

function playAudio() {
  if (audioBlob) {
    const audioUrl = URL.createObjectURL(audioBlob)
    audioPlayerRef.value.src = audioUrl
    audioPlayerRef.value.play()
  }
}

</script>

<template>
  <div>
    <button @click="startRecording">开始录制</button>
    <button @click="finishRecording">结束录制</button>
    <button @click="playAudio">播放录音</button>

    <audio ref="audioPlayerRef" controls></audio>
  </div>
</template>
