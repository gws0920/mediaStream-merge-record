<script setup>
import { ref } from 'vue'
import { VideoStreamMerger } from 'video-stream-merger'

const video = ref(null)
const recording = ref(false)
const running = ref(false)
let screen = null
let camera = null
let merger = null
let recorder = null
// 获取屏幕共享/摄像头，并 合并媒体流
const begin = async () => {``
  merger = new VideoStreamMerger({
    width: 680,   // Width of the output video
    height: 380,  // Height of the output video
    fps: 25,       // Video capture frames per second
    clearRect: true
  })
  // 获取屏幕共享
  try {
    screen = await navigator.mediaDevices.getDisplayMedia({ video: true, audio: true })
    merger.addStream(screen, {
      x: 0,
      y: 0,
      width: merger.width,
      height: merger.height
    })
  } catch (e) {
    console.error('出错了', e);
    screen = null
  }
  // 获取摄像头
  try {
    camera = await navigator.mediaDevices.getUserMedia({ video: true, audio: true })
    merger.addStream(camera, {
      x: merger.width - 100,
      y: merger.height - 100,
      width: 100,
      height: 100
    })
  } catch (e) {
    console.error('出错了', e);
    camera = null
  }

  merger.start()
  // 在video中播放
  const videoDom = document.querySelector('video')
  videoDom.srcObject = merger.result
  videoDom.play()
  running.value = true
}

// 停止
const stop = () => {
  stopStream(screen)
  stopStream(camera)
  merger.destroy()
  merger = null
  const videoDom = document.querySelector('video')
  videoDom.pause()
  running.value = false
  if (recording.value) {
    stopRecord()
  }
}

// 开始录制, 可通过time设置ondataavailable的回调间隔
const startRecord = (time) => {
  if (!merger) {
    return alert('请先点击“开始”')
  }
  recorder = new MediaRecorder(merger.result)
  recorder.ondataavailable = e => {
    download(e.data)
  }
  recorder.start(time > 0 ? time : undefined)
  recording.value = true
}

// 结束录制
const stopRecord = () => {
  recorder.stop()
  recording.value = false
}

// 下载
const download = (chunk) => {
  let url = URL.createObjectURL(chunk);
  var eleLink = document.createElement('a');
  eleLink.download = `video.webm`;
  eleLink.href = url
  document.body.appendChild(eleLink);
  eleLink.click();
  document.body.removeChild(eleLink);
}
const stopStream = stream => stream?.getTracks()?.forEach(track => track.stop())
</script>

<template>
  <div>
    <button @click="begin" v-if="!running">开始</button>
    <button @click="stop" v-else>停止</button>
    <button @click="startRecord" v-if="!recording">开始录制</button>
    <button @click="stopRecord" v-else>停止录制</button>
  </div>
  <h2>合并结果</h2>
  <video ref="video"></video>
</template>

<style scoped>
video {
  width: 680px;
  height: 380px;
  border: 1px solid;
}
</style>
