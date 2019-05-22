<template>
  <AIcon :type="iconType" class="audio-icon" @click="toggle" />
</template>

<script>
import { showErrorTip, isIE } from '@/utils/helpers'

const audios = [ ]

const pauseOtherAudios = (audioVm) => {
  audios.forEach((audio) => audioVm !== audio && audio.pause())
}

export default {
  name: 'AudioTest',
  data () {
    let audio = null

    // 如果是空串/null 之类的会报错
    audio = new Audio(undefined)
    audio.preload = false
    audio.autoplay = false
    audio.addEventListener('play', this.onPlay)
    audio.addEventListener('pause', this.onPause)
    // audio.addEventListener('error', this.onError)
    return {
      audio,
      playing: false
    }
  },
  computed: {
    iconType () {
      return (!this.playing) ? 'play-circle' : 'pause-circle'
    }
  },
  props: {
    src: { type: String, default: undefined }
  },
  watch: {
    src (nu, old) {
      this.audio.src = nu
      if (nu) {
        this.safePlay()
      }
    }
  },
  methods: {
    onPlay () { this.playing = true },
    onPause () { this.playing = false },
    onError () {
      showErrorTip(new Error('音频异常'))
      this.audio.pause()
      this.playing = false
    },
    safePlay () {
      pauseOtherAudios()
      if (this.audio) {
        const playPromise = this.audio.play()
        // IE下play的返回值不是promise
        if (playPromise) {
          playPromise.then(null, this.onError)
        }
      }
    },
    pause () {
      this.audio && this.audio.pause()
    },
    toggle () {
      if (isIE) {
        this.$warning({
          title: 'ie浏览器不支持音频测听，需要测听请下载',
          onOk () { }
        })
        return
      }
      if (!this.src) {
        this.$emit('getSrc')
        return
      }
      if (!this.audio.src) {
        this.audio.src = this.src
      }
      if (this.playing) {
        this.pause()
      } else {
        this.safePlay()
      }
    },
    dispose () {
      if (this.audio) {
        this.audio.removeEventListener('play', this.onPlay)
        this.audio.removeEventListener('pause', this.onPause)
        // this.audio.removeEventListener('error', this.onError)
        this.audio.pause()
        this.audio.remove()
        this.audio = null
      }
    }
  },
  mounted () {
    audios.push(this)
  },
  beforeDestroy () {
    const indexInAudios = audios.indexOf(this)
    if (indexInAudios !== -1) {
      audios.splice(indexInAudios, 1)
    }
    this.dispose()
  }
}
</script>

<style scoped lang="less">
  @import "~@/styles/vars.less";

  .audio-icon {
    font-size: 16px;
    color: @primary-color;
    cursor: pointer;
  }
</style>
