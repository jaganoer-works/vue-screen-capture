<script setup>
import { ref } from 'vue';
const stream = ref(null);

const startCapture = async () => {
    console.log("startCapture!");
    try {
        stream.value = await navigator.mediaDevices.getDisplayMedia({
            video: true,
            audio: false,
        });
        dumpOptionsInfo();
    } catch (err) {
        console.error(err);
    }
    return stream;
}

const stopCapture = () => {
    console.log('stopCapture');
    const tracks = stream.value.getTracks();

    tracks.forEach(track => track.stop());
    stream.value = null;
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
        <video :srcObject.prop="stream" autoplay></video>
    </div>
</template>
