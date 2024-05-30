<script setup>
import {onMounted, ref, toRefs} from 'vue'
import { fabric } from 'fabric'

const props = defineProps({
  imageData: {
    type: Object,
    default: () => {}
  }
})

const { imageData } = toRefs(props)

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
const initializeCanvas = () => {
  canvasRef.value = new fabric.Canvas(canvasDomRef.value)
  console.log(canvasDomRef.value)
  console.log(imageData.value, 'imagedata')
  // canvas 设置宽高为operaBox的宽高
  canvasRef.value.setWidth(operaBoxRef.value.offsetWidth)
  canvasRef.value.setHeight(operaBoxRef.value.offsetHeight)

  fabric.Image.fromURL(imageData.value.url, img => {
    // img.scaleToWidth(canvasRef.value.width)
    // 图片自适应缩放至宽或高等于canvas的宽或高

    canvasRef.value.add(img)
    imageRef.value = img
    fitImageToCanvas()
  })

  // 启用缩放功能
  canvasRef.value.on('mouse:wheel', (event) => {
    const delta = event.e.deltaY;
    const zoomRatio = 0.1;
    const pointer = canvasRef.value.getPointer(event.e);

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

    const offsetX = pointer.x - (pointer.x - imageRef.value.left) * (imageWidth / (imageWidth - zoom * zoomRatio));
    const offsetY = pointer.y - (pointer.y - imageRef.value.top) * (imageHeight / (imageHeight - zoom * zoomRatio));
    console.log(offsetX, offsetY, 'offsetX,offsetY')
    imageRef.value.left = offsetX;
    imageRef.value.top = offsetY;

    canvasRef.value.requestRenderAll();
  })
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