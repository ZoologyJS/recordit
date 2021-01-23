<template>
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
</template>

<script>
export default {
    name: "TitleCard"
}
</script>

<style scoped>
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