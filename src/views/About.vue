<template>
  <div class="about">
    <h1>This is an about page</h1>
    <p @click="addCount">{{count}}</p>
    <p>{{name1}}</p>
    <canvas id="myCanvas" width="500" height="40" />
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted, watch, toRef, reactive } from '@vue/composition-api'

export default defineComponent({
  props: {
    name: {
      type: String,
      required: false,
      default: ''
    }
  },
  setup (props) {
    const name1 = toRef(props, 'name')

    /**
     * canvas 跑马灯逻辑
     */
    let myCanvas: HTMLCanvasElement | null = null
    const count = ref(1)
    const addCount = (): void => {
      count.value++
    }
    let requestAnimationId: number | null = null
    let ctxObj = reactive({
      ctx: ({} as CanvasRenderingContext2D),
      windowW: 0,
      x: 0,
      img: ({} as HTMLImageElement),
      txt: ''
    })
    const speedX = 2
    const draw = () => {
      if (!ctxObj.ctx) return null
      const { ctx, img, windowW, txt } = ctxObj
      ctx.clearRect(0, 0, (myCanvas as HTMLCanvasElement).width, (myCanvas as HTMLCanvasElement).height)
      ctx.beginPath()
      ctx.drawImage(img, 7.5, 7.5, 25, 25)
      ctx.fillStyle = 'yellow'
      ctx.fillRect(0, 0, 40, 40)
      ctx.beginPath()
      ctx.globalCompositeOperation = 'destination-over'
      ctxObj.x -= speedX
      if (ctxObj.x <= -windowW) {
        ctxObj.x = windowW
        addCount()
      }
      ctx.font = '20px serif'
      ctx.fillStyle = 'red'
      ctx.textAlign = 'start'
      ctx.textBaseline = 'middle'
      ctx.fillText(txt, ctxObj.x, 20)
      ctx.closePath()
      requestAnimationId = window.requestAnimationFrame(draw)
    }

    const newNotice = (): void => {
      count.value = 1
      ctxObj.txt = '我是一条新的弹幕...'
      draw()
    }

    watch(count, (newVal: number): void => {
      if (newVal > 3) {
        window.cancelAnimationFrame(requestAnimationId as number)
        setTimeout(() => {
          newNotice()
        }, 1000)
      }
    })

    onMounted(() => {
      myCanvas = document.querySelector('#myCanvas')
      const ctx: CanvasRenderingContext2D | null = (myCanvas as HTMLCanvasElement).getContext('2d', { alpha: false })
      const txt = '我是一条弹幕'
      const bodyD: HTMLElement | null = document.querySelector('body')
      const windowW: number = (bodyD as HTMLElement).clientWidth
      const x: number = windowW;
      (myCanvas as HTMLCanvasElement).setAttribute('width', x + '')
      const img: HTMLImageElement = new Image()
      img.src = require('./tip.png')
      ctxObj = {
        ctx: (ctx as CanvasRenderingContext2D),
        windowW,
        x,
        img,
        txt
      }
      draw()
    })

    return {
      count,
      addCount,
      name1
    }
  }
})
</script>

<style lang="scss" scoped>
#myCanvas{
  border: 1px solid rgb(255, 0, 0);
  background: yellow;
}
</style>
