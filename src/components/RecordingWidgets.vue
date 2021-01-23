<template>
    <div class="main">
        <div class="button-container">
            <v-btn 
                class="capture-btn"
                ref="capture-btn"
                :disabled="disableStopped"
                @click="captureHandler">
                <v-icon small color="green">mdi-record</v-icon> Record
            </v-btn> 
            <v-btn 
                class="stop-btn"
                ref="stop-btn" 
                :disabled="disableRecording" 
                @click="stopHandler">
                <v-icon small color="red">mdi-stop</v-icon> Stop
            </v-btn>
            <v-btn 
                class="trash-btn"
                ref="trash-btn" 
                :disabled="disableTrash" 
                @click="trashHandler">
                <v-icon medium color="grey">mdi-trash-can</v-icon>
            </v-btn>
            <a class="download-button" ref="downloadButton">
                <v-icon small>mdi-cloud</v-icon> Download
            </a>
        </div>
        <br>
        <span class="text">Preview:</span>
        <br>
        <video 
            class="preview" 
            ref="preview"
            muted 
            autoplay>
        </video>
        <br>
        <span class="text">Your Recording:</span>
        <br>
        <video 
            class="recording" 
            ref="recording" 
            controls>
        </video>
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
         disableRecording: true,
         disableStopped: false,
         disableTrash: true
    }),
    methods: {
        // Captures the user interactions of the browser, sends those frames to our
        // startRecording handler, which then returns values that are converted to a Blob,
        // given a unique URL, and finally is downloadable via the download button
        captureHandler() {
            const previewSource = this.$refs.preview;
            const downloadButton = this.$refs.downloadButton;

            navigator.mediaDevices.getDisplayMedia(this.mediaConfig)
            .then(stream => {
                previewSource.srcObject = stream;
                downloadButton.href = stream;
                previewSource.captureStream = previewSource.captureStream || previewSource.mozCaptureStream;
                return new Promise(resolve => previewSource.onplaying = resolve);
            })
            .then(() => this.startRecording(previewSource.captureStream(), this.RECORDING_TIME_MS))
            .then (recordedChunks => {
                let recordedBlob = new Blob(recordedChunks, { type: "video/mp4" });
                this.$refs.recording.src = URL.createObjectURL(recordedBlob);
                downloadButton.href = this.$refs.recording.src;
                downloadButton.download = "recording.mp4";
            })
        },
        // Handles the recording of screen media after client "record" button click
        startRecording(stream) {
            let recorder = new MediaRecorder(stream);
            let data = [];

            this.disableRecording = false;
            this.disableStopped = true;

            recorder.ondataavailable = event => data.push(event.data);
            recorder.start();

            let stopped = new Promise((resolve, reject) => {
                recorder.onstop = resolve;
                recorder.onerror = event => reject(event.name);
            });

            // Long polling to check when screen recording has been stopped by client
            let recorded = new Promise(resolve => setInterval(() => {
                if (!this.disableStopped) resolve();
            }, 200))
            .then(() => recorder.state == "recording" && recorder.stop());

            // Return with an array of the recorded data upon promise resolutions
            return Promise.all([ stopped, recorded ]).then(() => data);
        },
        // Handles the stopping of recording after client "stop" button click
        stopHandler() {
            const tracks = this.$refs.preview.srcObject.getTracks();

            this.disableRecording = true;
            this.disableStopped = false;
            this.disableTrash = false;

            tracks.forEach(track => { track.stop(); })
            this.$refs.preview.srcObject = null;
            this.$refs.downloadButton.style.display = "inline";
        },
        trashHandler() {
            this.$refs.preview.src = null;
            this.$refs.downloadButton.href = null;
            this.$refs.downloadButton.download = null;
            this.$refs.downloadButton.style.display = "none";
            this.$refs.recording.src = null;
            this.disableTrash = true;
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

    .button-container {
        max-width: 600px;
    }

    video {
        border: 1px solid grey;
        margin: 5px;
        height: 300px;
        width: 500px;
        background-color: #f2f2f2;
    }

    .text {
        color: white;
    }

    a {
        text-decoration: none;
        float: right;
        display: none;
    }


</style>