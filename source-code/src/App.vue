<template>
  <div id="app">
    <img class="background-img" :class="{'background-img-open':open}" src="./assets/timg.jpeg">
    <div class="disc" @click="discClick" ref="disc">
      <div class="song-img" ref="rotate"></div>
    </div>
    <div class="lrc" :class="{'lrc-hide':open}" ref="lrc">
      <div v-for="(item,index) in lines" :class="currentLine == index?lrcActive:''">
        {{item.text}}
      </div>
    </div>
    <audio ref="audio" src="../static/jing.mp3" @playing="playing" autoplay loop></audio>
  </div>
</template>

<script>
  import lrc from './assets/song-lrc'
  import Rebound from 'rebound'

  export default {
    name: 'App',
    data() {
      return {
        lines: [],
        currentLine: 0,
        time: '00:00.00',
        lrcSpring: {},
        timer: {},

        open: false,

        discSpring: {}
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

      let that = this

      let springSystem = new Rebound.SpringSystem()
      this.lrcSpring = springSystem.createSpring(50, 3)
      this.lrcSpring.addListener({
        onSpringUpdate: function (spring) {
          let val = spring.getCurrentValue()
          that.$refs.lrc.scrollTop = val
        }
      })

      this.discSpring = springSystem.createSpring(50, 6)
      this.discSpring.addListener({
        onSpringUpdate: function (spring) {
          let val = spring.getCurrentValue()
          // console.log('val-->' + val)
          let discVal = Rebound.MathUtil.mapValueInRange(val, 0, 1, 10, 35)
          that.$refs.disc.style.marginLeft = discVal + '%'
          let scaleVal = Rebound.MathUtil.mapValueInRange(val, 0, 1, 0.7, 1)
          that.$refs.disc.style.transform = "scale(" + scaleVal + ") rotate(45deg)"
        }
      })

      this.facLrcs()

      // 转动头像
      this.$refs.rotate.style.animationPlayState = 'running'
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
        let startTime = new Date()
        clearInterval(this.timer)
        this.timer = setInterval(function () {
          let currentTime = new Date()
          let diff = new Date(currentTime - startTime)
          let ms = parseInt(diff.getMilliseconds() / 10)
          that.time = `${that.pad(diff.getMinutes())}:${that.pad(diff.getSeconds())}.${that.pad(ms)}`
        }, 80)
      },
      pad(x) {
        return (x < 10) ? ("0" + x) : x;
      },
      playing() {
        console.log('playing-->')
        this.run()
      },
      discClick() {
        this.open = !this.open
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
        if (val > 5) {
          this.lrcSpring.setEndValue((val - 5) * 15)
          // this.$refs.lrc.scrollTop = (val - 5) * 20
          // Velocity(this.$refs.lrc, {scroll: (val - 4) * 20}, {duration: 1000})
        } else {
          this.lrcSpring.setEndValue(0)
          // this.$refs.lrc.scrollTop = 0
          // Velocity(this.$refs.lrc, {scroll: 0}, {duration: 1000})
        }
      },
      open(val) {
        console.log('open-->' + val)
        if (val) {
          this.discSpring.setEndValue(1)
        } else {
          this.discSpring.setEndValue(0)
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
    margin-left -10%
    background-size: cover
    z-index: -1
    left: 0
    top: 0
    right: 0
    bottom: 0
    filter: blur(6px)
    transition 1s ease-in-out

  .background-img-open
    transform scale(1.2)
    filter: blur(20px)

  .disc
    position relative
    margin-top 10%
    margin-left 10%
    width disc-width
    height disc-width
    border-radius disc-width
    transform scale(0.7) rotate(45deg)
    background black
    background-image radial-gradient(5em 30em ellipse, #fff, #000)
    border 2px solid #131313
    box-shadow 0 0 0 10px #343935
    opacity 0.7

  .disc-open
    margin-top 15%
    margin-left 30%
    transform scale(1) rotate(45deg)

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
    width 40%
    height 50%
    color white
    transition 1s ease-in-out

  .lrc-hide
    right 2%
    opacity 0

  .lrc-active
    color red
    font-size 20px
    transition 1s ease-in-out

</style>
