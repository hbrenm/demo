<script setup>
import {onMounted, ref, reactive, watch} from 'vue'
const props = defineProps({
  imageData: {
    type: Object,
    default: () => {}
  }
})

const imagePath = ref('')
const imageAreaRef = ref()
const imageItemRef = ref()

const imagePosition = reactive({
  x1: 0,
  x2: 0,
  y1: 0,
  y2: 0
})


const setImageAttributes = () => {
  const { x1, y1, x2, y2 } = imagePosition
  imageAreaRef.value.style.width = `${x2 - x1}px`
  imageAreaRef.value.style.height = `${y2 - y1}px`
  imageItemRef.value.style.left = `-${x1}px`
  imageItemRef.value.style.top = `-${y1}px`
}

const handlePageMounted = () => {
  console.log(props.imageData.path)
  imagePath.value = props.imageData.path
  const { crop } = props.imageData
  const [x1, y1] = crop[0]
  const [x2, y2] = crop[1]
  imagePosition.x1 = x1
  imagePosition.y1 = y1
  imagePosition.x2 = x2
  imagePosition.y2 = y2
  setImageAttributes()
}
watch(
  () => props.imageData,
  () => {
    if (props.imageData && props.imageData.path) {
      handlePageMounted()
    }
  },
  { immediate: true }
)
onMounted(() => {
})

</script>

<template>
  <div class="image-area" ref="imageAreaRef">
    <img class="image-item" ref="imageItemRef" :src="imagePath" alt="">
  </div>
</template>

<style lang="less" scoped>
.image-area {
  position: relative;
  overflow: hidden;
  transform-origin: left top;
}

.image-item {
  position: absolute;
}
</style>