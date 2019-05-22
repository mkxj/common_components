<template>
  <div class="audio-player" :class="{ 'disabled': oprDisabled }" >
    <template v-if="!isIE">
      <AIcon :type="iconType" class="controller" @click="toggle" />
      <AProgress :percent="percent" :show-info="false" @click="handleProgressbarClick" :success-percent="-10" />
    </template>
    <template v-else>
      {{ NOT_SUPPORT_TIP }}
    </template>
  </div>
</template>
<script>
import { isIE } from '@cm/utils/helpers'
import { showErrorTip } from '@cm/utils/app-tip'

export default {
  props: {
    src: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  data () {
    let audio = null

    if (!isIE) {
      // 如果是空串/null 之类的会报错
      audio = new Audio(this.src || undefined)
      audio.preload = false
      audio.autoplay = false
      audio.addEventListener('canplay', () => { this.canPlay = true })
      audio.addEventListener('play', () => {
        this.playing = true
        this.updatePercent()
        this.startDetecting()
      })
      audio.addEventListener('pause', () => {
        this.playing = false
        this.stopDetecting()
      })
      audio.addEventListener('ended', () => {
        this.updatePercent()
      })
      audio.addEventListener('error', () => {
        showErrorTip(new Error('播放异常'))
      })
    }

    return {
      isIE,
      audio,
      canPlay: false,
      playing: false,
      percent: 0,
      detectTimer: null,
      NOT_SUPPORT_TIP: 'IE 不支持播放器，请使用 Chrome 等新版本浏览器。'
    }
  },
  computed: {
    oprDisabled () {
      return this.disabled || !this.src
    },
    iconType () {
      return (this.oprDisabled || !this.playing) ? 'play-circle' : 'pause-circle'
    }
  },
  watch: {
    src (nu, old) {
      if (!isIE) {
        this.audio.src = nu
        if (this.audio) {
          this.audio.pause()
          this.percent = 0
        }
      }
    }
  },
  methods: {
    updatePercent () {
      const x = this.audio.currentTime
      const max = this.audio.duration
      // eslint-disable-next-line
      if (max !== max) {
        this.percent = 0
        return
      }
      this.percent = (x * 100 / max) // .toFixed(2)
    },
    check () {
      if (this.oprDisabled) {
        // showErrorTip(new Error('不能操作'))
        return false
      }
      if (isIE) {
        showErrorTip(new Error(this.NOT_SUPPORT_TIP))
        return false
      }

      return !!this.audio
    },
    play () {
      const valid = this.check()
      if (valid) {
        this.audio.play()
      }
    },
    pause () {
      const valid = this.check()
      if (valid) {
        this.audio.pause()
      }
    },
    toggle () {
      const valid = this.check()
      if (!valid) {
        return
      }
      if (this.playing) {
        this.audio.pause()
      } else {
        this.audio.play()
      }
    },
    startDetecting () {
      this.detectTimer = setInterval(() => {
        this.updatePercent()
      }, 1000)
    },
    stopDetecting () {
      if (this.detectTimer) {
        clearInterval(this.detectTimer)
        this.detectTimer = null
      }
    },
    // 懒得自己写了，就用这个 AProgress 吧。。
    handleProgressbarClick ({ target, offsetX }) {
      if (this.oprDisabled || !this.audio) {
        return
      }
      if (this.audio.duration !== this.audio.duration) {
        return
      }
      if (!target.classList.contains('ant-progress-inner') && !target.classList.contains('ant-progress-bg')) {
        return
      }
      this.audio.currentTime = Math.floor(offsetX * this.audio.duration / target.offsetWidth)
      this.percent = Math.floor(offsetX * 100 / target.offsetWidth)
    },
    dispose () {
      if (!isIE && this.audio) {
        this.audio.pause()
        this.audio.remove()
        this.audio = null
      }
    }
  },
  beforeDestroy () {
    this.dispose()
  }
}
</script>

<style scoped lang="less">
  @import "~@cm/styles/vars.less";

  .audio-player {
    display: flex;

    &.disabled {
      cursor: not-allowed;

      .controller {
        color: @text-color;
        cursor: not-allowed;
      }
    }

    .controller {
      margin-right: 10px;
      font-size: 26px;
      color: @primary-color;
      cursor: pointer;
    }
  }
</style>
