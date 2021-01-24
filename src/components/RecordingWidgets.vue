<template>
    <div class="main">
        <div class="button-container">
            <v-dialog
            v-model="dialog"
            width="500"
            >
                <template v-slot:activator="{ on, attrs }">
                    <v-btn
                    class="capture-btn"
                    :disabled="disableStopped"
                    v-bind="attrs"
                    v-on="on"
                    >
                    <v-icon small color="green">mdi-record</v-icon> Record 
                    </v-btn>
                </template>

                <v-card>
                    <v-card-title>
                    Recording Source
                    </v-card-title>

                    <v-card-text>
                    Select what media you would like to record.
                    </v-card-text>

                    <v-divider></v-divider>

                    <!-- <v-card-actions> -->
                    <v-spacer></v-spacer>
                    <div class="d-flex">
                        <v-checkbox 
                            v-model="soundCheck"
                            class="mr-auto ml-4"
                            label="Record audio"
                        ></v-checkbox>
                        <v-btn
                            class="mt-3 mr-2"
                            color="primary"
                            @click="dialog = false; captureHandler('webcam')"
                        >
                            Webcam
                        </v-btn>
                        <v-btn
                            class="mt-3 mr-4"
                            color="primary"
                            @click="dialog = false; captureHandler('screen')"
                        >
                            Screen
                        </v-btn>
                    </div>
                    <!-- </v-card-actions> -->
                </v-card>
            </v-dialog>
            <v-btn 
                class="stop-btn"
                :disabled="disableRecording" 
                @click="stopHandler(); snackbar = false">
                <v-icon small color="red">mdi-stop</v-icon> Stop
            </v-btn>
            <v-btn 
                class="trash-btn"
                :disabled="disableTrash" 
                @click="trashHandler">
                <v-icon medium color="grey">mdi-trash-can</v-icon>
            </v-btn>
            <a class="download-button" ref="downloadButton">
                <v-icon class="mr-1" small>mdi-cloud-download</v-icon>Download
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

        <v-snackbar
        v-model="snackbar"
        timeout="-1"
        color="red"
        content-class="text-center"
        >
        Recording in progress...
        </v-snackbar>
    </div>
</template>

<script>
export default {
    name: "RecordingWidgets",
    data: () => ({
        dialog: false,
        disableRecording: true,
        disableStopped: false,
        disableTrash: true,
        soundCheck: false,
        snackbar: false
    }),
    methods: {
        // Captures the user interactions of the browser, sends those frames to our
        // startRecording handler, which then returns values that are converted to a Blob,
        // given a unique URL, and finally is downloadable via the download button
        captureHandler(source) {
            const previewSource = this.$refs.preview;
            const downloadButton = this.$refs.downloadButton;
            const configz = {       
                video: {
                    cursor: "always"
                },
                audio: this.soundCheck
            }
            console.log
            const mediaSource = source == "screen" 
                ? navigator.mediaDevices.getDisplayMedia(configz) 
                : navigator.mediaDevices.getUserMedia(configz);
            
            mediaSource.then(stream => {
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
            this.snackbar = true;

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
            this.$refs.downloadButton.style.display = "flex";
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

    .stop-btn, .capture-btn, .trash-btn, a {
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
        width: 600px;
        background-color: #f2f2f2;
    }

    .download-button {
        align-items: center;
        justify-content: center;
    }

    .text {
        color: white;
    }

    /* .v-snack__content {
        text-align: center !important
    } */

    a {
        text-decoration: none;
        float: right;
        display: none;
    }


</style>