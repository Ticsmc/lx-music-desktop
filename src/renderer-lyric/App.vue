<template lang="pug">
#container(:class="[theme, lrcConfig.isLock ? 'lock' : null]")
  #main
    transition(enter-active-class="animated-fast fadeIn" leave-active-class="animated-fast fadeOut")
      .control-bar(v-show="!lrcConfig.isLock")
        core-control-bar(:lrcConfig="lrcConfig" :themes="themeList")
    core-lyric(:lrcConfig="lrcConfig" :isPlayLxlrc="isPlayLxlrc" :isShowLyricTranslation="isShowLyricTranslation")
  div.resize-left(@mousedown.self="handleMouseDown('left', $event)" @touchstart.self="handleTouchDown('left', $event)")
  div.resize-top(@mousedown.self="handleMouseDown('top', $event)" @touchstart.self="handleTouchDown('top', $event)")
  div.resize-right(@mousedown.self="handleMouseDown('right', $event)" @touchstart.self="handleTouchDown('right', $event)")
  div.resize-bottom(@mousedown.self="handleMouseDown('bottom', $event)" @touchstart.self="handleTouchDown('bottom', $event)")
  div.resize-top-left(@mousedown.self="handleMouseDown('top-left', $event)" @touchstart.self="handleTouchDown('top-left', $event)")
  div.resize-top-right(@mousedown.self="handleMouseDown('top-right', $event)" @touchstart.self="handleTouchDown('top-right', $event)")
  div.resize-bottom-left(@mousedown.self="handleMouseDown('bottom-left', $event)" @touchstart.self="handleTouchDown('bottom-left', $event)")
  div.resize-bottom-right(@mousedown.self="handleMouseDown('bottom-right', $event)" @touchstart.self="handleTouchDown('bottom-right', $event)")
  core-icons
</template>

<script>
import { rendererOn, rendererInvoke, rendererSend, NAMES } from '../common/ipc'

window.ELECTRON_DISABLE_SECURITY_WARNINGS = process.env.ELECTRON_DISABLE_SECURITY_WARNINGS

export default {
  data() {
    return {
      resize: {
        origin: null,
        msDownX: 0,
        msDownY: 0,
      },
      lrcConfig: {
        enable: false,
        isLock: true,
        isAlwaysOnTop: false,
        width: 600,
        height: 700,
        x: -1,
        y: -1,
        theme: 0,
        style: {
          font: '',
          fontSize: 125,
          opacity: 80,
          isZoomActiveLrc: true,
        },
      },
      isShowLyricTranslation: true,
      isPlayLxlrc: true,
      themeList: [
        {
          id: 0,
          className: 'green',
        },
        {
          id: 1,
          className: 'yellow',
        },
        {
          id: 2,
          className: 'blue',
        },
        {
          id: 3,
          className: 'red',
        },
        {
          id: 4,
          className: 'pink',
        },
        {
          id: 5,
          className: 'purple',
        },
        {
          id: 6,
          className: 'orange',
        },
        {
          id: 7,
          className: 'grey',
        },
        {
          id: 8,
          className: 'ming',
        },
        {
          id: 9,
          className: 'blue2',
        },
      ],
    }
  },
  computed: {
    theme() {
      let theme = this.themeList.find(t => t.id == this.lrcConfig.theme) || this.themeList[0]
      return theme.className
    },
  },
  created() {
    rendererOn(NAMES.winLyric.set_lyric_config, (event, config) => this.handleUpdateConfig(config))
    rendererInvoke(NAMES.winLyric.get_lyric_config).then(config => this.handleUpdateConfig(config))
  },
  mounted() {
    document.addEventListener('mousemove', this.handleMouseMove)
    document.addEventListener('mouseup', this.handleMouseUp)
  },
  beforeDestroy() {
    document.removeEventListener('mousemove', this.handleMouseMove)
    document.removeEventListener('mouseup', this.handleMouseUp)
  },
  methods: {
    handleUpdateConfig({ config, languageId, isShowLyricTranslation, isPlayLxlrc }) {
      this.lrcConfig = config
      this.isShowLyricTranslation = isShowLyricTranslation
      this.isPlayLxlrc = isPlayLxlrc
      if (this.$i18n.locale !== languageId && languageId != null) this.$i18n.locale = languageId
    },
    handleDown(origin, clientX, clientY) {
      this.handleMouseUp()
      this.resize.origin = origin
      this.resize.msDownX = clientX
      this.resize.msDownY = clientY
    },
    handleMouseUp() {
      this.resize.origin = null
    },
    handleMouseDown(origin, event) {
      this.handleDown(origin, event.clientX, event.clientY)
    },
    handleTouchDown(origin, event) {
      if (event.changedTouches.length) {
        const touch = event.changedTouches[0]
        this.handleDown(origin, touch.clientX, touch.clientY)
      }
    },
    handleMouseMove(event) {
      this.handleMove(event.clientX, event.clientY)
    },
    handleTouchMove(event) {
      if (event.changedTouches.length) {
        const touch = event.changedTouches[0]
        this.handleMove(touch.clientX, touch.clientY)
      }
    },
    handleMove(clientX, clientY) {
      if (!this.resize.origin) return
      // if (!event.target.classList.contains('resize-' + this.resize.origin)) return
      // console.log(event.target)
      let bounds = {
        w: 0,
        h: 0,
      }
      let temp
      switch (this.resize.origin) {
        case 'left':
          temp = clientX - this.resize.msDownX
          bounds.w = -temp
          bounds.x = temp
          break
        case 'right':
          bounds.w = clientX - this.resize.msDownX
          this.resize.msDownX += bounds.w
          break
        case 'top':
          temp = clientY - this.resize.msDownY
          bounds.y = temp
          bounds.h = -temp
          break
        case 'bottom':
          bounds.h = clientY - this.resize.msDownY
          this.resize.msDownY += bounds.h
          break
        case 'top-left':
          temp = clientX - this.resize.msDownX
          bounds.w = -temp
          bounds.x = temp
          temp = clientY - this.resize.msDownY
          bounds.y = temp
          bounds.h = -temp
          break
        case 'top-right':
          temp = clientY - this.resize.msDownY
          bounds.y = temp
          bounds.h = -temp
          bounds.w = clientX - this.resize.msDownX
          this.resize.msDownX += bounds.w
          break
        case 'bottom-left':
          temp = clientX - this.resize.msDownX
          bounds.w = -temp
          bounds.x = temp
          bounds.h = clientY - this.resize.msDownY
          this.resize.msDownY += bounds.h
          break
        case 'bottom-right':
          bounds.w = clientX - this.resize.msDownX
          this.resize.msDownX += bounds.w
          bounds.h = clientY - this.resize.msDownY
          this.resize.msDownY += bounds.h
          break
      }
      // console.log(bounds)
      bounds.w = window.innerWidth + bounds.w
      bounds.h = window.innerHeight + bounds.h
      rendererSend(NAMES.winLyric.set_win_bounds, bounds)
    },
    // handleMouseOver() {
    //   // this.handleMouseUp()
    // },
  },
}
</script>

<style lang="less">
@import './assets/styles/index.less';
@import './assets/styles/layout.less';

body {
  user-select: none;
  height: 100vh;
  box-sizing: border-box;
  color: #fff;
  opacity: .8;
}

#container {
  padding: 8px;
  box-sizing: border-box;
  height: 100%;
  &.lock {
    #main {
      background-color: transparent;
    }
  }
}

.resize-left {
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  width: 10px;
  cursor: ew-resize;
  // background-color: rgba(0, 0, 0, 1);
}
.resize-right {
  position: absolute;
  right: 0;
  top: 0;
  height: 100%;
  width: 10px;
  cursor: ew-resize;
}
.resize-top {
  position: absolute;
  left: 0;
  top: 0;
  height: 10px;
  width: 100%;
  cursor: ns-resize;
}
.resize-bottom {
  position: absolute;
  left: 0;
  bottom: 0;
  height: 10px;
  width: 100%;
  cursor: ns-resize;
}
.resize-top-left {
  position: absolute;
  left: 0;
  top: 0;
  width: 14px;
  height: 14px;
  cursor: nwse-resize;
  // background-color: rgba(0, 0, 0, 1);
}
.resize-top-right {
  position: absolute;
  right: 0;
  top: 0;
  width: 14px;
  height: 14px;
  cursor: nesw-resize;
  // background-color: rgba(0, 0, 0, 1);
}
.resize-top-left {
  position: absolute;
  left: 0;
  top: 0;
  width: 14px;
  height: 14px;
  cursor: nwse-resize;
  // background-color: rgba(0, 0, 0, 1);
}
.resize-bottom-left {
  position: absolute;
  left: 0;
  bottom: 0;
  width: 14px;
  height: 14px;
  cursor: nesw-resize;
  // background-color: rgba(0, 0, 0, 1);
}
.resize-bottom-right {
  position: absolute;
  right: 0;
  bottom: 0;
  width: 14px;
  height: 14px;
  cursor: nwse-resize;
  // background-color: rgba(0, 0, 0, 1);
}

#main {
  position: relative;
  box-sizing: border-box;
  height: 100%;
  transition: background-color @transition-theme;
  min-height: 0;
  border-radius: @radius-border;
  overflow: hidden;
  background-color: rgba(0, 0, 0, .2);

  &:hover {
    .control-bar {
      opacity: 1;
    }
  }
}

.control-bar {
  position: absolute;
  border-top-left-radius: @radius-border;
  border-top-right-radius: @radius-border;
  overflow: hidden;
  top: 0;
  left: 0;
  width: 100%;
  opacity: 0;
  transition: opacity @transition-theme;
  z-index: 1;
}
</style>
