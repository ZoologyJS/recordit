<template>
  <v-app>
    <v-main>
      <!-- <div class="cont">
        <v-color-picker
          class="ma-2"
          canvas-height="300"
          @update:color="getColor"
        ></v-color-picker>
      </div> -->
      <Container />
    </v-main>
  </v-app>
</template>

<script>
// import { debounce } from 'vue-debounce'
import Container from './components/Container';

export default {
  name: 'App',
  components: { Container },
  data() {
    return {
      // currColor: null
    }
  },
  methods: {
    // getColor: function(e) {
    //     console.log(e)
    //     const {r, g, b} = e.rgba;
    //     this.currColor = `rgb(${r}, ${g},${b})`;
    //     this.changeBody(this.currColor);
    //     console.log(this.currColor)
    // },
    // changeBody: function(color) {
    //   document.querySelector(".cont").style.backgroundColor = color;
    // }
  }
};
</script>

<style>
  body > #app {
    background-color: #1f2e42;
  }
</style>
