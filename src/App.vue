<script setup>
import { ref, onMounted } from "vue";
import axios from "axios"

const cameraActive = ref(true)
const video = ref(null);
const canvas = ref(null)
const dataURL = ref(null)
const uploadStatus = ref(null)
const prediction = ref(null)

const leaves = {
    'Sorbus aucuparia': {
        name: 'Rönn',
        img: 'https://upload.wikimedia.org/wikipedia/commons/9/9a/R%C3%B6nnb%C3%A4r.jpg',
        description: 'Rönn, Sorbus aucuparia, är ett träd inom familjen rosväxter. Dess löv, blomning om våren och röda bär på hösten är karaktäristiska och lätt identifierbara tecken.'
    }
}

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

function fileUpload() {
    const d = new Date();
    let time = d.getTime();
    let data = new FormData();
    canvas.value.toBlob(function (blob) {
        data.append('file', blob, time.toString() + '.png');
        uploadStatus.value = 'uploading'

        axios
            .post('http://localhost:5000/api', data, {
                headers: {
                    'Content-Type': 'multipart/form-data',
                },
            })
            .then(res => {
                console.log(res)
                prediction.value = res.data
                uploadStatus.value = 'done'
            })
            .error(err => {
                console.error(err)
                uploadStatus.value = 'error'
            });
    });
}


async function testAPI() {
    try {
        const res = await axios.get('http://localhost:5000/api')
        console.log(res.data.response)
    } catch (err) {
        console.error(err)
    }
}

onMounted(async () => {
    await openCamera();
})

</script>

<template>
    <div v-if="!uploadStatus" class="absolute bottom-0">
        <video autoplay ref="video"></video>
        <canvas class="hidden" ref="canvas"></canvas>
    </div>
    <div v-if="cameraActive" class="w-full h-48 absolute bottom-0 flex justify-center items-center">
        <div @click="takePicture" class="bg-white h-16 w-16 rounded-full active:bg-gray-200"></div>
    </div>
    <div v-if="!cameraActive && !uploadStatus"
        class="absolute bottom-0 bg-white w-full h-4/5 flex flex-col rounded-t-3xl px-8 py-4 items-center">
        <h1 class="font-semibold text-2xl mb-3">Är du nöjd med bilden?</h1>
        <i-mdi-close @click="cameraActive = true" class="absolute text-gray-600 h-10 w-10 absolute top-3 right-3">
        </i-mdi-close>
        <img :src="dataURL" class="rounded-lg h-3/4" />
        <div class="flex justify-around w-full">
            <i-material-symbols-replay-circle-filled-rounded @click="cameraActive = true"
                class="h-20 w-20 text-red-500 rounded-full active:text-red-600">
            </i-material-symbols-replay-circle-filled-rounded>
            <i-material-symbols-check-circle-rounded class="h-20 w-20 text-green-500 rounded-full active:text-green-600"
                @click="fileUpload">
            </i-material-symbols-check-circle-rounded>
        </div>
    </div>
    <div v-if="uploadStatus" class="min-h-screen w-full flex justify-center bg-white p-4">
        <i-mdi-loading v-if="uploadStatus === 'loading'" class="animate-spin h-16 w-16 text-green-500"></i-mdi-loading>
        <div v-elif="uploadStatus === 'done'" class="flex flex-col items-center">
            <h1 class="font-semibold text-3xl">{{ leaves[prediction.leaf_species].name }}</h1>
            <p class="text-green-500 text-xl font-semibold">{{prediction.confidence.toFixed(1)}}%</p>
            <p class="text-gray-700">{{ prediction.leaf_species }}</p>
            <img class="rounded-lg" alt="" :src="leaves[prediction.leaf_species].img" />
            <p class="text-gray-700">{{ leaves[prediction.leaf_species].description }}</p>
            <br />
            <p @click="uploadStatus = null; cameraActive = true; openCamera()" class="text-green-500 underline">Gör om</p>
        </div>
    </div>
</template>
