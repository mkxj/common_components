<template>
  <div class="clipboard" :class="{'clipboard-disabled': disabled}">
    <slot v-if="!disabled">
      <ATooltip :title="tip" placement="bottom" @visibleChange="handleVisibleChange">
        <AIcon class="clipboard-trigger" type="copy" />
      </ATooltip>
    </slot>
  </div>
</template>

<script>
import Clipboard from 'clipboard'

const defaultTip = '点击复制名称'
const successTip = '复制成功!'
const errorTip = '复制失败!'

export default {
  name: 'Clipboard',
  props: {
    text: {
      type: [String, Function],
      default () {
        return (trigger) => trigger.dataset['clipboardText']
      }
    },
    target: {
      type: [String, Function],
      default () { return '' } // If not function, clipboard will use defaultTarget
    },
    action: {
      type: [String, Function],
      default () { return 'copy' }
    },
    disabled: { type: Boolean, default: false },
    onSuccess: { type: Function },
    onError: { type: Function }
  },
  data () {
    // 以下用法基于 https://clipboardjs.com/ Advanced Usage
    // text 优先级高于 target
    const textFn = typeof this.text === 'string' ? () => this.text : this.text
    let targetFn = null
    const vm = this
    if (typeof this.target === 'function') {
      targetFn = this.target
    } else {
      targetFn = function (/* trigger */) {
        let selector = vm.target === '' ? vm.$el.dataset['clipboardTarget'] : vm.target
        if (selector) {
          return document.querySelector(selector)
        }
      }
    }
    const actionFn = typeof this.action === 'string' ? () => this.action : this.action
    return {
      textFn,
      targetFn,
      actionFn,
      clipboard: null,
      tip: defaultTip
    }
  },
  methods: {
    handleSuccess (e) {
      this.tip = successTip
      if (this.onSuccess) {
        this.onSuccess(e)
      }
    },
    handleError (e) {
      this.tip = errorTip
      if (this.onError) {
        this.onError(e)
      }
    },
    handleVisibleChange (visible) {
      if (visible) {
        this.tip = defaultTip // 不能 if (!visible) 来判断，因为还有动画，$nextTick 也不行
      }
    }
  },
  mounted () {
    const clipboard = new Clipboard(this.$el, {
      text: this.textFn,
      target: this.targetFn,
      action: this.actionFn
    })
    clipboard.on('success', this.handleSuccess)
    clipboard.on('error', this.handleError)
    this.clipboard = clipboard
  },
  beforeDestroy () {
    this.clipboard && this.clipboard.destroy()
  }
}
</script>

<style scoped lang="less">
  @import "~@cm/styles/vars.less";

  .clipboard {
    display: flex;
    align-items: center;
    cursor: pointer;
    user-select: none;
  
    &.clipboard-disabled {
      cursor: not-allowed;

      .clipboard-trigger {
        color: rgba(0, 0, 0, 0.25);
      }
    }
    
    &-trigger {
      font-size: 18px;
      color: @text-color; // @primary-color;
    }
  }
</style>
