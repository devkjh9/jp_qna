<template>
  <div class="div-canvas-wrapper">
    <canvas
      id="c-canvas"
      v-bind:width="cWidth"
      v-bind:height="cHeight"
      v-on:mousedown="down"
      v-on:mousemove="move"
      v-on:mouseout="up"
      v-on:mouseup="up"
    ></canvas>
    <div class="div-canvas-button-wrapper">
      <button class="btn-info" v-on:click="clear">초기화</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CCanvas',
  data: function () {
    return {
      cWidth: 480,
      cHeight: 320,
      cObject: null,
      isDrawing: false,
      isDown: false,
      prevX: 0,
      prevY: 0,
      currX: 0,
      currY: 0,
      strokeStyle: 'blank',
      lineWidth: 2
    }
  },
  mounted () {
    this.init()
  },
  methods: {
    init () {
      this.$parent.$on('clearCanvas', () => {
        this.clear()
      })
      this.cObject = document.getElementById('c-canvas')
      if (this.isMobile()) {
        this.cWidth = window.outerWidth - 2
        const that = this
        this.cObject.addEventListener('touchmove', function (e) {
          that.move(e)
        }, false)
        this.cObject.addEventListener('touchstart', function (e) {
          e.preventDefault()
          that.down(e)
        }, false)
        this.cObject.addEventListener('touchend', function (e) {
          that.up()
        }, false)
      }
    },
    getCtx () {
      return this.cObject.getContext('2d')
    },
    clear () {
      this.getCtx().clearRect(0, 0, this.cWidth, this.cHeight)
      this.isDrawing = false
      this.isDown = false
      this.prevX = 0
      this.prevY = 0
      this.currX = 0
      this.currY = 0
    },
    down: function (e) {
      if (e.touches) {
        this.currX = e.touches[0].clientX - this.cObject.offsetLeft
        this.currY = e.touches[0].clientY - this.cObject.offsetTop + window.scrollY
      } else {
        this.currX = e.clientX - this.cObject.offsetLeft
        this.currY = e.clientY - this.cObject.offsetTop + window.scrollY
      }
      this.isDrawing = true
    },
    up () {
      this.isDrawing = false
    },
    move: function (e) {
      if (this.isDrawing) {
        this.prevX = this.currX
        this.prevY = this.currY
        if (e.touches) {
          this.currX = e.touches[0].clientX - this.cObject.offsetLeft
          this.currY = e.touches[0].clientY - this.cObject.offsetTop + window.scrollY
        } else {
          this.currX = e.clientX - this.cObject.offsetLeft
          this.currY = e.clientY - this.cObject.offsetTop + window.scrollY
        }
        const ctx = this.getCtx()
        ctx.beginPath()
        ctx.moveTo(this.prevX, this.prevY)
        ctx.lineTo(this.currX, this.currY)
        ctx.strokeStyle = this.strokeStyle
        ctx.lineWidth = this.lineWidth
        ctx.stroke()
        ctx.closePath()
      }
    },
    isMobile () {
      if (navigator.userAgent.match(/Android/i) ||
            navigator.userAgent.match(/webOS/i) ||
            navigator.userAgent.match(/iPhone/i) ||
            navigator.userAgent.match(/iPad/i) ||
            navigator.userAgent.match(/iPod/i) ||
            navigator.userAgent.match(/BlackBerry/i) ||
            navigator.userAgent.match(/Windows Phone/i)
      ) {
        return true
      } else {
        return false
      }
    }
  }
}
</script>

<style scoped>
  canvas {
    background: #ffffff;
    border: 1px solid #ccc;
    margin-top: 15px
  }
</style>
