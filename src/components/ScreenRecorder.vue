<script setup>
import { ref } from 'vue';
let stream = ref(null);
let mediaRecorder = ref(null);
let src = ref(null);
let isAutoPlay = ref(false);
let blob = ref([]);

const startCapture = async () => {
    console.log("startCapture!");

    try {
        stream.value = await navigator.mediaDevices.getDisplayMedia({
            video: true,
            audio: false,
        });
        mediaRecorder.value = new MediaRecorder(stream.value, { mimeType: 'video/webm; codecs=vp8' });
        blob.value = [];
        handleDataAvailable();
        mediaRecorder.value.start();
        console.log("mediaRecorder.state: " + mediaRecorder.value.state);
        console.log("recorder started");
        dumpOptionsInfo();

    } catch (err) {
        console.error(err);
    }
}

const handleDataAvailable = () => {
    mediaRecorder.value.ondataavailable = event => {
        console.log('handleDataAvailable', event);
        if (event.data && event.data.size > 0) {
            blob.value.push(event.data);
        }
    };
}

const stopCapture = () => {
    mediaRecorder.value.stop();
    stream.value.getTracks().forEach(StreamData => StreamData.stop());
    console.log(mediaRecorder.value.state);
    console.log("recorder stopped");
}

const startPlay = () => {
    const superBuffer = new Blob(blob.value);
    const url = window.URL.createObjectURL(superBuffer);
    src.value = url
    // this.isDisabled = false
    isAutoPlay.value = true
    setTimeout(() => {
        window.URL.revokeObjectURL(url);
    }, 100);
}

const downloadMovie = () => {
    const downloadBlob = new Blob(blob.value);
    const link = document.createElement("a");
    const url = window.URL.createObjectURL(downloadBlob);
    link.href = url
    link.download = Date.now() + '.webm';
    link.click();
    setTimeout(() => {
        window.URL.revokeObjectURL(url);
    }, 100);
}

const dumpOptionsInfo = () => {
    const videoTrack = stream.value.getVideoTracks()[0];

    console.info("Track settings:");
    console.info(JSON.stringify(videoTrack.getSettings(), null, 2));
    console.info("Track constraints:");
    console.info(JSON.stringify(videoTrack.getConstraints(), null, 2));
}

</script>

<template>
    <div>
        <button @click="startCapture">開始</button>
        <button @click="stopCapture">停止</button>
        <button @click="startPlay">再生</button>
        <button @click="downloadMovie">ダウンロード</button>
        <div>
            <video controls="true" :src="src" :autoplay="isAutoPlay"></video>
        </div>
    </div>
</template>
