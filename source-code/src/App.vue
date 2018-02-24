<template>
  <div id="app">
    <img class="background-img" src="./assets/timg.jpeg">
    <audio ref="audio" src="../static/jing.mp3" autoplay loop></audio>
    <home/>
  </div>
</template>

<script>
  import Home from './components/Home'

  export default {
    name: 'App',
    components: {
      'home': Home
    },
    mounted() {
      let audio = this.$refs.audio
      audio.volume = 0.5
    }
  }
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
  }

  body {
    margin: 0;
    padding: 0;
    height: 100%;
    overflow: hidden;
  }

  .background-img {
    position: absolute;
    width: 100%;
    height: 100%;
    background-size: cover;
    z-index: -1;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    filter: blur(20px);
  }
</style>
