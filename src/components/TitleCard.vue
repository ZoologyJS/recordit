<template>
    <div class="full-cont">
        <v-card class="title-cont">
            <v-row align="center" justify="center" style="margin-bottom: 50px">
                <span class="main-title">RecordIt</span>
                <br>
                <span class="under-title"><i>Capture content in seconds!</i></span>
            </v-row>
            <v-divider />
            <v-row class="mt-5" align="center" justify="center" style="padding: 20px;">
                <div class="another-cont">
                    <p class="directions">
                        <b>Here's how to start:</b>
                        <v-list-item>
                            <v-list-item-content>
                                1. Select the "Record" button.
                            </v-list-item-content>
                        </v-list-item>
                        <v-list-item>
                            <v-list-item-content>
                                2. Choose the source you want to record.
                            </v-list-item-content>
                        </v-list-item>
                        <v-list-item>
                            <v-list-item-content>
                                3. When you're done recording, select the "Stop button".
                            </v-list-item-content>
                        </v-list-item>
                        <v-list-item>
                            <v-list-item-content>
                                4. Select "Download" to download the recording to your machine.
                            </v-list-item-content>
                        </v-list-item>
                    </p> 
                </div>
            </v-row>
        </v-card>
    </div>
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
        height: 93vh;
        padding: 20px;
        border-radius: 20px;
        margin-right: 50px;
    }

    .directions {
        font-size: 20px;
    }

    .another-cont {
        text-align: left;
        padding: 0px 40px
    }
</style>