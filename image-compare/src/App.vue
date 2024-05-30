<script setup>

import {ref} from 'vue'
import ImageCanvas from './components/image-canvas.vue'
const imageList = ref([
  {
    url: 'http://127.0.0.1:8080/wallhaven-83dq9k.jpg'
  }
])

</script>

<template>
  <div class="compare-page">
    <template v-for="(item, index) in imageList">
      <image-canvas :imageData="item"></image-canvas>
    </template>
  </div>
</template>

<style lang="less" scoped>
.compare-page {
  width: 100vw;
  height: 100vh;
  display: flex;
  overflow: hidden;
}
</style>