<template>
  <video
    :src="video"
    controls
  />
  <br>
  <button 
    @click="transcode" 
    :disabled="message == '트랜스코딩..'"
  >
    시작
  </button>
  <p>{{ message }}</p>
</template>

<script>
import { createFFmpeg, fetchFile } from "@ffmpeg/ffmpeg";
import { defineComponent, ref } from "vue";

// const targetFile = 'flame.avi';
const targetFile = 'IMG_6696.MOV';

export default defineComponent({
  name: "App",
  setup() {
    // app state
    const ffmpeg = createFFmpeg({
      log: true,
    });
    const message = ref("시작 버튼을 클릭하면 트랜스코딩이 시작됩니다.");
    let video = ref(null);
    const file =
      process.env.NODE_ENV === "production"
        ? `/vue-app/${targetFile}`
        : `/${targetFile}`;
    // methods
    async function transcode() {
      message.value = "ffmeg-core.js 로딩..";

      const startTime = new Date().getTime();
      
      await ffmpeg.load();
      message.value = "트랜스코딩..";
      ffmpeg.FS("writeFile", "tmp.avi", await fetchFile(file));
      await ffmpeg.run("-i", "tmp.avi", "-s", "1280x720", "tmp.mp4");

      const endTime = new Date().getTime();

      message.value = `1280x720(MP4) 트랜스코딩 완료, 걸린 시간: ${endTime - startTime}ms`;
      const data = ffmpeg.FS("readFile", "tmp.mp4");
      video.value = URL.createObjectURL(
        new Blob([data.buffer], { type: "video/mp4" })
      );
    }
    return {
      video,
      message,
      transcode,
    };
  },
});
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
