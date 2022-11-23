<script setup>
import { ref, onMounted } from "vue";

const video = ref(null);
const canvas = ref(null)
const dataURL = ref(null)
const photo = ref(null)

const emit = defineEmits(['takePicture'])

async function openCamera() {
    try {
        const stream = await navigator.mediaDevices.getUserMedia({
            video: {
                facingMode: "environment",
                height: window.innerWidth,
                width: window.innerHeight,
            },
            audio: false,
        });
        video.value.srcObject = stream;
        canvas.value.width = window.innerWidth
        canvas.value.height = window.innerHeight
    } catch (error) {
        console.error(error);
    }
}

function takePicture() {
    const context = canvas.value.getContext('2d');
    context.drawImage(video.value, 0, 0, canvas.value.width, canvas.value.height);
    dataURL.value = canvas.value.toDataURL('image/png')
    cameraActive.value = false
}

onMounted(async () => {
    await openCamera();
})

</script>

<template>
    <div class="absolute bottom-0">
        <video autoplay ref="video"></video>
        <canvas class="hidden" ref="canvas"></canvas>
    </div>
    <div class="w-full h-48 absolute bottom-0 flex justify-center items-center">
        <div @click="takePicture" class="bg-white h-16 w-16 rounded-full active:bg-gray-200"></div>
    </div>
</template>
