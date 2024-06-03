<script setup>
import {onMounted, onUnmounted, ref, toRefs} from 'vue'
import { EventBus } from '../uitils/eventBus.js'
import {fabric} from 'fabric'

const props = defineProps({
  imageData: {
    type: Object,
    default: () => {
    }
  },
  imageIndex: {
    type: Number,
    default: 0
  }
})
const lastImageLeft = ref(0)
const lastImageTop = ref(0)

const handleMouseDown = () => {

}

const emitEvent = (eventName, data) => {
  EventBus.emit(eventName, data)
}

const handleDoomZoom = (data) => {
  const { e } = data
  const delta = e.e.deltaY;
  const zoomRatio = 0.1;
  console.log(props.imageIndex, e.e)
  const mockEvent = {
    clientX: e.e.offsetX + canvasDomRef.value.getBoundingClientRect().left,
    clientY: e.e.offsetY + canvasDomRef.value.getBoundingClientRect().top,
    preventDefault: () => {},
    stopPropagation: () => {}
  }
  const pointer = canvasRef.value.getPointer(mockEvent);
  const beforeZoom = imageRef.value.scaleX;
  let zoom = imageRef.value.scaleX;

  if (delta > 0) {
    zoom -= zoomRatio;
  } else {
    zoom += zoomRatio;
  }

  imageRef.value.scaleX = zoom;
  imageRef.value.scaleY = zoom;
  console.log(props.imageIndex, pointer.x)

  // 计算鼠标指针在图片上的位置 以便缩放后保持鼠标指针在图片上的位置不变
  const offsetX = (pointer.x - imageRef.value.left) / beforeZoom - (pointer.x - imageRef.value.left) / zoom
  const offsetY = (pointer.y - imageRef.value.top) / beforeZoom - (pointer.y - imageRef.value.top) / zoom
  imageRef.value.left -= offsetX * zoom;
  imageRef.value.top -= offsetY * zoom;
  canvasRef.value.requestRenderAll();
}

const handleDoMoving = (data) => {
  const { deltaX, deltaY } = data
  imageRef.value.left = lastImageLeft.value + deltaX
  imageRef.value.top = lastImageTop.value + deltaY
  canvasRef.value.requestRenderAll()
}

const eventHandler = {
  'doZoom': (data) => {
    handleDoomZoom(data)
  },
  'doMoving': (data) => {
    handleDoMoving(data)
  },
  'doMousseDown': () => {
    lastImageLeft.value = imageRef.value.left
    lastImageTop.value = imageRef.value.top
  }
}
for (const [eventName, handler] of Object.entries(eventHandler)) {
  EventBus.on(eventName, handler)
}

onUnmounted(() => {
  for (const [eventName, handler] of Object.entries(eventHandler)) {
    EventBus.off(eventName, handler)
  }
})

const {imageData} = toRefs(props)

const canvasRef = ref()
const canvasDomRef = ref()
const operaBoxRef = ref()
const imageRef = ref()

function fitImageToCanvas() {
  const canvasWidth = canvasRef.value.width;
  const canvasHeight = canvasRef.value.height;
  const imageWidth = imageRef.value.width * imageRef.value.scaleX;
  const imageHeight = imageRef.value.height * imageRef.value.scaleY;

  const scaleX = canvasWidth / imageWidth;
  const scaleY = canvasHeight / imageHeight;
  const scale = Math.min(scaleX, scaleY);

  imageRef.value.scaleX *= scale;
  imageRef.value.scaleY *= scale;

  const offsetX = (canvasWidth - imageWidth * scale) / 2;
  const offsetY = (canvasHeight - imageHeight * scale) / 2;

  imageRef.value.left += offsetX;
  imageRef.value.top += offsetY;
  canvasRef.value.requestRenderAll();
}

const canvasLoadImage = async (url) => {
  return new Promise((resolve, reject) => {
    fabric.Image.fromURL(url, img => {
      resolve(img)
    })
  })
}

const initializeCanvas = async () => {
  canvasRef.value = new fabric.Canvas(canvasDomRef.value)
  // canvas 设置宽高为operaBox的宽高
  canvasRef.value.setWidth(operaBoxRef.value.offsetWidth)
  canvasRef.value.setHeight(operaBoxRef.value.offsetHeight)

  imageRef.value = await canvasLoadImage(imageData.value.url)
  // 图片添加移动事件
  imageRef.value.on('mousedown', (event) => {
    const { clientX, clientY } = event.e
    imageRef.value.set({ lastMouseX: clientX, lastMouseY: clientY})
    emitEvent('doMousseDown', null)
  })
  imageRef.value.on('moving', (event) => {
    console.log('image moving')
    const { clientX, clientY } = event.e
    const { lastMouseX, lastMouseY } = imageRef.value
    const deltaX = clientX - lastMouseX
    const deltaY = clientY - lastMouseY
    emitEvent('doMoving', { deltaX, deltaY })
  })
  // imageRef.value.ad
  canvasRef.value.add(imageRef.value)
  // 隐藏控制点
  imageRef.value.setControlsVisibility({
    ml: false,
    mb: false,
    mr: false,
    mt: false,
    tr: false,
    tl: false,
    bl: false,
    br: false,
    mtr: false
  })
  fitImageToCanvas()

  // 启用缩放功能
  canvasRef.value.on('mouse:wheel', (e) => {
    emitEvent('doZoom', {
      e,
    })

  })
}

const handleTest = () => {
  console.log(canvasDomRef.value.getBoundingClientRect().left)
}

onMounted(() => {
  initializeCanvas()
})

</script>
<template>
  <div class="drawing-content">
    <div class="operation-box" ref="operaBoxRef">
      <canvas class="image-canvas" ref="canvasDomRef"></canvas>
    </div>

    <div class="tools">
      <button @click="handleTest">click</button>
    </div>
  </div>
</template>

<style lang="less" scoped>
.drawing-content {
  width: 50%;
  height: 50%;
  position: relative;
  background-color: #f5f5f5;
  border: 1px solid red;

  .tools {
    position: absolute;
    top: 0;
    left: 0;
  }

  .operation-box {
    width: 100%;
    height: 100%;
  }
}

.image-canvas {
  width: 100%;
  height: 100%;
}
</style>