<script setup>
import {onMounted, ref} from 'vue'
import CropImage from './components/crop-image.vue'

const jsonData = ref(null)
const selectedImage = ref({})


onMounted(() => {
  jsonData.value = imageObject
  selectedImage.value = imageObject.images[0]
})

</script>

<template>
  <div>
    <crop-image style="transform: scale(0.1)" :image-data="selectedImage"></crop-image>
  </div>
</template>