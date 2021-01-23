<template>
    <div class="main">
        <div class="button-container">
            <v-btn class="capture-btn" :disabled="isStopped" @click="captureHandler"><v-icon small color="green">mdi-record</v-icon> Record</v-btn> 
            <v-btn class="stop-btn" :disabled="isRecording" @click="stopHandler"><v-icon small color="red">mdi-stop</v-icon> Stop</v-btn>
            <v-btn class="trash-btn" :disabled="canTrash" @click="trashHandler"><v-icon medium color="grey">mdi-trash-can</v-icon></v-btn>
            <a class="download-button"><v-icon small>mdi-cloud</v-icon> Download</a>
        </div>
        <br>
        <span class="text">Preview:</span>
        <br>
        <video class="preview" muted autoplay></video>
        <br>
        <span class="text">Your Recording:</span>
        <br>
        <video class="recording" controls></video>
    </div>
</template>

<script>
export default {
    name: "RecordingWidgets",
    data: () => ({
        // Configurations for capturing screen media
        mediaConfig: {
            video: {
                cursor: "always"
            },
            audio: false
        },
         isRecording: true,
         isStopped: false,
         canTrash: true
    }),
    methods: {
        // Captures the user interactions of the browser, sends those frames to our
        // startRecording handler, which then returns values that are converted to a Blob,
        // given a unique URL, and finally is downloadable via the download button
        captureHandler() {
            const videoSource = document.querySelector(".preview");
            const downloadButton = document.querySelector(".download-button");

            navigator.mediaDevices.getDisplayMedia(this.mediaConfig)
            .then(stream => {
                videoSource.srcObject = stream;
                downloadButton.href = stream;
                videoSource.captureStream = videoSource.captureStream || videoSource.mozCaptureStream;
                return new Promise(resolve => videoSource.onplaying = resolve);
            })
            .then(() => this.startRecording(videoSource.captureStream(), this.RECORDING_TIME_MS))
            .then (recordedChunks => {
                const recording = document.querySelector(".recording");
                let recordedBlob = new Blob(recordedChunks, { type: "video/mp4" });
                recording.src = URL.createObjectURL(recordedBlob);
                downloadButton.href = recording.src;
                downloadButton.download = "recording.mp4";
            })
        },
        // Handles the recording of screen media after client "record" button click
        startRecording(stream) {
            let recorder = new MediaRecorder(stream);
            let data = [];

            this.isRecording = false;
            this.isStopped = true;

            recorder.ondataavailable = event => data.push(event.data);
            recorder.start();

            let stopped = new Promise((resolve, reject) => {
                recorder.onstop = resolve;
                recorder.onerror = event => reject(event.name);
            });

            // Long polling to check when screen recording has been stopped by client
            let recorded = new Promise(resolve => setInterval(() => {
                if (!this.isStopped) resolve();
            }, 200))
            .then(() => recorder.state == "recording" && recorder.stop());

            // Return with an array of the recorded data upon promise resolutions
            return Promise.all([ stopped, recorded ]).then(() => data);
        },
        // Handles the stopping of recording after client "stop" button click
        stopHandler() {
            this.isRecording = true;
            this.isStopped = false;
            this.canTrash = false;
            const videoSource = document.querySelector(".preview");
            let tracks = videoSource.srcObject.getTracks();

            tracks.forEach(track => { track.stop(); })
            videoSource.srcObject = null;
            document.querySelector("a").style.display = "inline"
        },
        trashHandler() {
            const videoSource = document.querySelector(".preview");
            const downloadButton = document.querySelector(".download-button");
            const recording = document.querySelector(".recording");

            videoSource.src = null;
            downloadButton.href = null;
            downloadButton.download = null;
            downloadButton.style.display = "none";
            recording.src = null;
            this.canTrash = true;
        }
    }
}
</script>

<style scoped>
    @import url('https://fonts.googleapis.com/css2?family=Alice&display=swap');

    button, a {
        border: 1px solid grey;
        border-radius: 5px;
        padding: 5px 15px;
        margin: 5px;
        background-color: #f2f2f2;
    }

    video {
        border: 1px solid grey;
        margin: 5px;
        height: 300px;
        width: 600px;
        background-color: #f2f2f2;
    }

    .text {
        color: white;
    }

    .button-container {
        max-width: 600px;
    }

    a {
        text-decoration: none;
        float: right;
        display: none;
    }


</style>