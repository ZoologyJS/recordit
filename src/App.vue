<template>
  <v-app>
    <v-main>
      <v-container class="cont">
          <v-row>
              <v-col>
                  <TitleCard />
              </v-col>
              <v-col>
                  <RecordingWidgets />
              </v-col>
          </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import RecordingWidgets from './components/RecordingWidgets';
import TitleCard from './components/TitleCard';

export default {
  name: 'App',
  components: { RecordingWidgets, TitleCard },
  data() {
    return {
    }
  },
  methods: {
  }
};
</script>

<style>
  body > #app {
    background-color: #1f2e42;
  }
</style>
