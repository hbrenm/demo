<script setup>
import {onMounted, ref, toRefs} from 'vue'
import {fabric} from 'fabric'

const props = defineProps({
  imageData: {
    type: Object,
    default: () => {
    }
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
  canvasRef.value.on('mouse:wheel', (event) => {
    const delta = event.e.deltaY;
    const zoomRatio = 0.1;
    const pointer = canvasRef.value.getPointer(event.e);
    const beforeZoom = imageRef.value.scaleX;
    let zoom = imageRef.value.scaleX;

    if (delta > 0) {
      zoom -= zoomRatio;
    } else {
      zoom += zoomRatio;
    }

    imageRef.value.scaleX = zoom;
    imageRef.value.scaleY = zoom;

    const imageWidth = imageRef.value.width * zoom;
    const imageHeight = imageRef.value.height * zoom;
    // 计算鼠标指针在图片上的位置 以便缩放后保持鼠标指针在图片上的位置不变
    const offsetX = (pointer.x - imageRef.value.left) / beforeZoom - (pointer.x - imageRef.value.left) / zoom
    const offsetY = (pointer.y - imageRef.value.top) / beforeZoom - (pointer.y - imageRef.value.top) / zoom
    console.log(offsetX)
    imageRef.value.left -= offsetX * zoom;
    imageRef.value.top -= offsetY * zoom;

    canvasRef.value.requestRenderAll();
  })
}

const getCursorPositonInImage = () => {
  const pointer = canvasRef.value.getPointer(event.e);
  const imageWidth = imageRef.value.width * imageRef.value.scaleX;
  const imageHeight = imageRef.value.height * imageRef.value.scaleY;
  const offsetX = (canvasRef.value.width - imageWidth) / 2;
  const offsetY = (canvasRef.value.height - imageHeight) / 2;
  const x = pointer.x - offsetX;
  const y = pointer.y - offsetY;
  return {x, y}
}


onMounted(() => {
  initializeCanvas()
})

const handleTest = () => {
  imageRef.value.left = -200;
  imageRef.value.top = -200;
  canvasRef.value.requestRenderAll()
}


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
  width: 60%;
  height: 70%;
  position: relative;
  background-color: #f5f5f5;

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