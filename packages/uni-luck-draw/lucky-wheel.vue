<template>
  <view v-if="isShow" class="lucky-box" :style="{ width: boxWidth + 'px', height: boxHeight + 'px' }">
    <canvas
      type="2d"
      id="lucky-wheel"
      canvas-id="lucky-wheel"
      :style="{ width: boxWidth + 'px', height: boxHeight + 'px' }"
    ></canvas>
    <!-- #ifdef APP-PLUS -->
    <view class="lucky-wheel-btn" @click="toPlay" :style="{ width: btnWidth + 'px', height: btnHeight + 'px' }"></view>
    <!-- #endif -->
    <!-- #ifndef APP-PLUS -->
    <cover-view class="lucky-wheel-btn" @click="toPlay" :style="{ width: btnWidth + 'px', height: btnHeight + 'px' }"></cover-view>
    <!-- #endif -->
    <view v-if="myLucky">
      <div class="lucky-imgs">
        <div v-for="(block, index) in blocks" :key="index">
          <div v-if="block.imgs">
            <image v-for="(img, i) in block.imgs" :key="i" :src="img.src" @load="e => imgBindload(e, 'blocks', index, i)"></image>
          </div>
        </div>
      </div>
      <div class="lucky-imgs">
        <div v-for="(prize, index) in prizes" :key="index">
          <div v-if="prize.imgs">
            <image v-for="(img, i) in prize.imgs" :key="i" :src="img.src" @load="e => imgBindload(e, 'prizes', index, i)"></image>
          </div>
        </div>
      </div>
      <div class="lucky-imgs">
        <div v-for="(btn, index) in buttons" :key="index">
          <div v-if="btn.imgs">
            <image v-for="(img, i) in btn.imgs" :key="i" :src="img.src" @load="e => imgBindload(e, 'buttons', index, i)"></image>
          </div>
        </div>
      </div>
    </view>
  </view>
</template>

<script>
  import { changeUnits, resolveImage } from './utils.js'
  import { LuckyWheel } from './interim-fix.js'
  export default {
    name: 'lucky-wheel',
    data () {
      return {
        myLucky: null,
        canvas: null,
        isShow: false,
        boxWidth: 100,
        boxHeight: 100,
        btnWidth: 0,
        btnHeight: 0,
        dpr: 1,
      }
    },
    props: {
      width: {
        type: String,
        default: '600rpx'
      },
      height: {
        type: String,
        default: '600rpx'
      },
      blocks: {
        type: Array,
        default: () => []
      },
      prizes: {
        type: Array,
        default: () => []
      },
      buttons: {
        type: Array,
        default: () => []
      },
      defaultConfig: {
        type: Object,
        default: () => ({})
      },
      defaultStyle: {
        type: Object,
        default: () => ({})
      },
    },
    mounted () {
      // #ifdef APP-PLUS
      console.error('该抽奖插件的最新版暂不支持app端, 请通过npm安装旧版本【npm i uni-luck-draw@1.3.9】')
      // #endif
      // #ifndef APP-PLUS
      this.initLucky()
      // #endif
    },
    watch: {
      blocks (newData) {
        this.myLucky && (this.myLucky.blocks = newData)
      },
      prizes (newData) {
        this.myLucky && (this.myLucky.prizes = newData)
      },
      buttons (newData) {
        this.myLucky && (this.myLucky.buttons = newData)
      },
      defaultStyle (newData) {
        this.myLucky && (this.myLucky.defaultStyle = newData)
      },
      defaultConfig (newData) {
        this.myLucky && (this.myLucky.defaultConfig = newData)
      },
    },
    methods: {
      async imgBindload (res, name, index, i) {
        const img = this[name][index].imgs[i]
        resolveImage(img, this.canvas)
      },
      initLucky () {
        this.boxWidth = changeUnits(this.width)
        this.boxHeight = changeUnits(this.height)
        this.isShow = true
        // 某些情况下获取不到 canvas
        this.$nextTick(() => {
          setTimeout(() => {
            this.draw()
          })
        })
      },
      draw () {
        const _this = this
        uni.createSelectorQuery().in(this).select('#lucky-wheel').fields({
          node: true, size: true
        }).exec((res) => {
          // #ifdef H5
          res[0].node = document.querySelector('#lucky-wheel canvas')
          // #endif
          if (!res[0] || !res[0].node) return console.error('lucky-canvas 获取不到 canvas 标签')
          const { node, width, height } = res[0]
          const canvas = this.canvas = node
          const ctx = this.ctx = canvas.getContext('2d')
          const dpr = this.dpr = uni.getSystemInfoSync().pixelRatio
          // #ifndef H5
          canvas.width = width * dpr
          canvas.height = height * dpr
          ctx.scale(dpr, dpr)
          // #endif
          const myLucky = this.myLucky = new LuckyWheel({
            // #ifdef H5
            flag: 'UNI-H5',
            // #endif
            // #ifdef MP
            flag: 'MP-WX',
            // #endif
            ctx,
            dpr,
            width,
            height,
            setTimeout,
            clearTimeout,
            setInterval,
            clearInterval,
            // #ifdef H5
            rAF: requestAnimationFrame,
            // #endif
            unitFunc: (num, unit) => changeUnits(num + unit),
            beforeCreate: function () {
              const Radius = Math.min(this.config.width, this.config.height) / 2
              // 设置坐标轴
              _this.Radius = Radius
              ctx.translate(Radius, Radius)
            },
            beforeInit: function () {
              // 重置坐标轴
              ctx.translate(-this.Radius, -this.Radius)
            },
            afterInit: function () {
              // 动态设置按钮
              _this.btnWidth = this.maxBtnRadius * 2
              _this.btnHeight = this.maxBtnRadius * 2
              _this.$forceUpdate()
            },
          }, {
            ...this.$props,
            start: function (...rest) {
              _this.$emit('start', ...rest)
            },
            end: (...rest) => {
              this.$emit('end', ...rest)
            },
          })
        })
      },
      toPlay (e) {
        this.myLucky.startCallback()
      },
      play (...rest) {
        this.myLucky.play(...rest)
      },
      stop (...rest) {
        this.myLucky.stop(...rest)
      },
    },
  }
</script>

<style scoped>
  .lucky-box {
    position: relative;
    overflow: hidden;
  }
  .lucky-box canvas {
    position: absolute;
    pointer-events: none;
    left: 0;
    top: 0;
  }
  .lucky-wheel-btn {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    background: rgba(0, 0, 0, 0);
    border-radius: 50%;
  }
  .lucky-imgs {
    width: 0;
    height: 0;
    visibility: hidden;
  }
</style>
