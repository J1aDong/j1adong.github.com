<template>
  <div id="app">
    <img class="background-img" :class="{'background-img-open':open}" src="./assets/jing.png">
    <div class="disc" @click="discClick" ref="disc">
      <div class="song-img" ref="rotate"></div>
    </div>
    <div class="lrc" :class="{'lrc-hide':open}" ref="lrc">
      <div v-for="(item,index) in lines" :class="currentLine == index?lrcActive:''">
        {{item.text}}
      </div>
    </div>
    <a class="mj-title" :class="{'mj-title-open':open}" href="https://github.com/J1aDong" target="_blank">
      <icon name="github" scale="1.5"></icon>
      J1aDong.github.com</a>
    <div class="home-wrapper" :class="{'home-wrapper-open':open}" @mouseenter="homeHoverFun(true)"
         @mouseleave="homeHoverFun(false)">
      <span class="home" :class="{'home-hover':homeHover}"></span>
      <icon class="home-icon" :class="{'home-icon-hover':homeHover}" name="chevron-circle-up" color="white"></icon>
      <a class="home-content" :class="{'home-content-hover':homeHover}" href="http://47.96.31.106:32771/#/"
         target="_blank">我的空间站</a>
    </div>
    <div class="content" :class="{'content-open':open}">第五十天，好好的活下去</div>
    <audio ref="audio" src="../static/jing.mp3" @playing="playing" autoplay loop></audio>
    <input class="voice" ref="voiceRange" type="range" v-model="voice" :max="100" min="0">
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

        discSpring: {},

        homeHover: false,

        voice: 50
      }
    },
    computed: {
      lrcActive() {
        return 'lrc-active'
      }
    },
    mounted() {
      // let audio = this.$refs.audio
      // audio.volume = 0.5

      this.voice = localStorage.getItem('voice') || 100

      this.$refs.voiceRange.style.background = `linear-gradient(to right, #eb7470, #eb7470 ${this.voice}%, white ${this.voice}%, white)`

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
      },
      homeHoverFun(val) {
        this.homeHover = val
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
          this.lrcSpring.setEndValue((val - 5) * 18)
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
      },
      voice() {
        console.log("voice-->" + this.voice)
        this.$refs.audio.volume = this.voice / 100
        this.$refs.voiceRange.style.background = `linear-gradient(to right, #eb7470, #eb7470 ${this.voice}%, white ${this.voice}%, white)`

        localStorage.setItem("voice", this.voice)
      }
    }
  }
</script>

<style lang="stylus" type="text/stylus">
  disc-width = 25rem
  head-icon-width = 15rem
  home-button-width = 6rem

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
    width 120%
    height auto
    margin-top -10%
    margin-left -10%
    background-size: cover
    left: 0
    top: 0
    right: 0
    bottom: 0
    filter: blur(10px)
    transition transform 2s ease-in-out

  .background-img-open
    transform scale(1.2)

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
    cursor pointer

  .disc-open
    margin-top 15%
    margin-left 30%
    transform scale(1) rotate(45deg)

  input[type=range]
    -webkit-appearance: none
    height: 8px
    border-radius: 10px
    background-color: #fff

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
    font-size 1.6rem
    transition 1s ease-in-out

  .content
    position absolute
    bottom -2rem
    width 100%
    color white
    opacity 0.1
    transition 1s ease-in-out

  .content-open
    bottom 3rem
    opacity 1
    font-size 1.5rem
    transform rotateX(-360deg)

  .mj-title
    position fixed
    color rgba(178, 178, 178, 0.98)
    bottom 2rem
    left 10rem
    text-decoration none
    font-size 2rem
    transition 0.5s ease-in-out

  .mj-title-open
    font-size 1rem

  .home-wrapper
    position relative
    top 400px
    background-color #2c3e50
    opacity 1
    transition 1s ease-in-out
    visibility hidden
    transform rotateY(-90deg)

  .home-wrapper-open
    opacity 1
    top 200px
    visibility visible
    transform rotateY(0)

  .home
    position absolute
    left 50% - (home-button-width/ 2)
    bottom 20%
    width home-button-width
    height home-button-width
    border-radius home-button-width
    background-color rgba(0, 0, 0, 0.25)
    cursor pointer
    box-shadow 0 0 10px 1px rgba(0, 0, 0, 0.55)
    transition 0.3s ease-in-out

  .home-icon
    position absolute
    width 3rem
    height 3rem
    border-radius 3rem
    bottom 1.5rem
    left 50% - 1.3rem
    transition 0.5s ease-in-out

  .home-icon-hover
    left 50% - 6rem
    transform rotate(90deg)

  .home-hover
    position absolute
    left 50% - (home-button-width/ 2) - 5rem
    bottom 22%
    width home-button-width + 10rem
    height home-button-width
    border-radius home-button-width
    background-color rgba(0, 0, 0, 0.45)
    box-shadow 0 0 100px 1px rgba(0, 0, 0, 0.55)
    text-align left
    line-height home-button-width
    cursor pointer

  .home-content
    position absolute
    left 50%
    height 3.5rem
    opacity 0
    bottom 10%
    text-decoration none
    color #ef0f4e
    transition 0.2s ease-in-out

  .home-content-hover
    opacity 1
    font-size 1.2rem

  .voice
    position fixed
    bottom 5rem
    right 10rem
    width 15rem


</style>
