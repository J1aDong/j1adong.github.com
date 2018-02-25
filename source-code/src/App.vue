<template>
  <div id="app">
    <img class="background-img" src="./assets/timg.jpeg">
    <div class="disc">
      <div class="song-img" ref="rotate"></div>
    </div>
    <div class="lrc" ref="lrc">
      <div v-for="(item,index) in lines" :class="currentLine == index?lrcActive:''" style="top: -50px">
        {{item.text}}
      </div>
    </div>
    <audio ref="audio" src="../static/jing.mp3" autoplay loop></audio>
  </div>
</template>

<script>
  import lrc from './assets/song-lrc'
  import Velocity from 'velocity-animate'

  export default {
    name: 'App',
    data() {
      return {
        lines: [],
        currentLine: 0,
        time: '00:00.00',
      }
    },
    computed: {
      lrcActive() {
        return 'lrc-active'
      }
    },
    mounted() {
      let audio = this.$refs.audio
      audio.volume = 0.5
      // console.log(lrc)
      this.run()
    },
    methods: {
      facLrcs() {
        let lines = lrc.split("\n");
        for (let k in lines) {
          let timeMatch = lines[k].match(/\[(\d+:\d+\.\d+)\]/);
          this.lines.push({
            time: timeMatch ? timeMatch.pop() : '',
            text: lines[k].replace(/^.+?\]/, '')
          })
        }
      },
      run: function () {
        let that = this
        this.facLrcs()
        let startTime = new Date()
        let timer = setInterval(function () {
          let currentTime = new Date()
          let diff = new Date(currentTime - startTime)
          let ms = parseInt(diff.getMilliseconds() / 10)
          that.time = `${that.pad(diff.getMinutes())}:${that.pad(diff.getSeconds())}.${that.pad(ms)}`
        }, 100)
      },
      pad(x) {
        return (x < 10) ? ("0" + x) : x;
      }
    },
    watch: {
      time(val) {
        let that = this;
        let num = parseFloat(val.replace(':', '.')) * 1000;

        that.lines.forEach(function (item, index) {
          if (item.time.indexOf(val.substr(0, 7)) === 0) {
            that.currentLine = index
            console.log('currentLine-->' + that.currentLine)
            return false
          }
        })
      },
      currentLine(val) {
        if (val > 4) {
          // this.$refs.lrc.scrollTop = (val - 4) * 20
          Velocity(this.$refs.lrc, "scrollTop", {duration: 500, offset: (val - 4) * 20})
        } else {
          // this.$refs.lrc.scrollTop = 0
          Velocity(this.$refs.lrc, "scrollTop", {duration: 500, offset: 0})
        }
      }
    }
  }
</script>

<style lang="stylus" type="text/stylus">
  disc-width = 25rem
  head-icon-width = 15rem

  #app
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;

  body
    margin: 0;
    padding: 0;
    height: 100%;
    overflow: hidden;

  .background-img
    position absolute
    width 150%
    height auto
    margin-top -10%
    margin-left -10%;
    background-size: cover
    z-index: -1;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    filter: blur(20px);

  .disc
    position relative
    margin-top 30% - 10rem
    margin-left 50% - 10rem
    width disc-width
    height disc-width
    border-radius disc-width
    transform rotate(45deg)
    background black
    background-image radial-gradient(5em 30em ellipse, #fff, #000)
    border 2px solid #131313
    box-shadow 0 0 0 10px #343935
    opacity 0.7

  @keyframes animations1
    0%
      transform: translate(-50%, -50%) rotate(0deg)
    50%
      transform: translate(-50%, -50%) rotate(180deg)
    100%
      transform: translate(-50%, -50%) rotate(360deg)

  .song-img
    position absolute
    top 50%
    left 50%
    transform-origin center center
    transform translate(-50%, -50%) rotate(0deg)
    width head-icon-width
    height head-icon-width
    background-size cover
    background-repeat no-repeat
    background-position center
    border: 2px solid #000
    border-radius: head-icon-width
    animation: animations1 10s linear infinite forwards
    animation-play-state: paused
    background-image url("./assets/head.jpeg")

  .lrc
    position absolute
    right 10%
    top 10%
    overflow auto
    width 20%
    height 50%
    color white

  .lrc-active
    color red
    font-size 20px
    transition 1s ease-in-out
</style>
