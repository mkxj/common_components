<template>
  <div id="parent">
    <AIcon :type="iconType" class="audio-icon" @click="toggle" />
    <iframe id="iframe" ref="iframe" src="/static/AudioTestIe.html" name="myFrame" frameborder="0"></iframe>
  </div>
</template>

<script>
const url = location.protocol + '//' + location.host
export const inBrowser = typeof window !== 'undefined'
export const UA = inBrowser && window.navigator.userAgent.toLowerCase()
export const isIE = UA && /msie|trident/.test(UA)

export default {
  data () {
    return {
      playing: false
    }
  },
  props: [ 'src' ],
  computed: {
    iconType () {
      return (!this.playing) ? 'play-circle' : 'pause-circle'
    }
  },
  watch: {
    src (nu, old) {
      if (nu) {
        this.play()
      } else {
        this.pause()
      }
    }

  },
  methods: {
    toggle () {
      if (!this.src) {
        this.$emit('getSrc')
        return
      }
      if (!this.playing) {
        this.play()
      } else {
        this.pause()
      }
    },
    play () {
      this.playing = true
      window.myFrame.playSound(url + this.src)
    },
    pause () {
      this.playing = false
      window.myFrame.playSound('')
    }
  },
  beforeDestroy () {
    this.pause()
  }
}
</script>

<style scoped>
  #parent {
    width: 18px;
    height: 18px;
    line-height: 18px;
    display: inline-block;
  }
  #iframe {
    width: 100%;
    height: 100%;
  }
  .audio-icon {
    font-size: 16px;
    color: #1890FF;
    cursor: pointer;
  }
</style>
