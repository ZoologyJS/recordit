<template>
    <v-container class="cont">
        <v-row>
            <v-col>
                <v-card class="title-cont">
                    <v-row align="center" justify="center" style="margin-bottom: 50px">
                        <span class="main-title">RecordIt</span>
                        <br>
                        <span class="under-title"><i>Capture your screen in seconds!</i></span>
                    </v-row>
                    <v-row align="center" justify="center" style="padding: 20px;">
                        <div class="another-cont">
                            <p class="directions">
                                <b>Here's how to start:</b>
                                <br>
                                1. Select the "Record" button.
                                <br>
                                2. Choose the source you want to record.
                                <br>
                                3. When you're done recording, select the "Stop button".
                                <br>
                                4. Select "Download" to download the recording to your machine.
                            </p> 
                        </div>
                    </v-row>
                </v-card>
            </v-col>
            <v-col>
                <div class="button-container">
                    <v-btn :disabled="isStopped" @click="captureHandler"><v-icon small color="green">mdi-record</v-icon> Record</v-btn> 
                    <v-btn :disabled="isRecording" @click="stopHandler"><v-icon small color="red">mdi-stop</v-icon> Stop</v-btn>
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
            </v-col>
        </v-row>
    </v-container>
</template>

<script>
export default {
    name: "Container",
    data: () => ({
        mediaConfig: {
            video: {
                cursor: "always"
            },
            audio: false
        },
         RECORDING_TIME_MS: 4000,
         isRecording: true,
         isStopped: false
    }),
    methods: {
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
                    downloadButton.download = "RecordedVideo.mp4";
                })
        },
        startRecording(stream, lengthInMS) {
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

            let recorded = this.wait(lengthInMS)
                .then(() => recorder.state == "recording" && recorder.stop());

            return Promise.all([
                stopped,
                recorded
            ])
            .then(() => data);
        },
        wait() {
            return new Promise(resolve => setInterval(() => {
                if (!this.isStopped) resolve();
            }, 200));
        },
        stopHandler() {
            this.isRecording = true;
            this.isStopped = false;
            const videoSource = document.querySelector(".preview");
            let tracks = videoSource.srcObject.getTracks();

            tracks.forEach(track => { track.stop(); })
            videoSource.srcObject = null;
            document.querySelector("a").style.display = "inline"
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

    .main-title {
        font-family: 'Alice', serif;
        font-size: 72px;
        color: rgb(26, 26, 26);
    }

    .under-title {
        font-family: 'Alice', serif;
        font-size: 24px;
        color: rgb(26, 26, 26);
        
    }

    .title-cont {
        height: 95vh;
        background-color: #f2f2f2;
        padding: 20px;
        border-radius: 20px;
        margin-right: 50px;
        /* text-align: center; */
    }

    .directions {
        font-size: 20px;
    }

    .another-cont {
        text-align: left;
        padding: 0px 40px
    }

</style>