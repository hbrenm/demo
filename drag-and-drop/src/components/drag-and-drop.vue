<script setup>
import { ref, onMounted, reactive } from 'vue'

const images = ref([
  'http://127.0.0.1:8080/wallhaven-83dq9k.jpg',
  'http://127.0.0.1:8080/dea8dd861fb130f55ff85e2e50cd245a.jpg',
  'http://127.0.0.1:8080/cd947ce52287bd399f1c50a2af87f8d7.jpg',
  'http://127.0.0.1:8080/cae5548fb6c3bb761df999c543cd468c.jpg'
])

const selectedImages = ref([]);
const droppedImages = ref([]);

const toggleSelect = (image) => {
  const index = selectedImages.value.indexOf(image);
  if (index === -1) {
    selectedImages.value.push(image);
  } else {
    selectedImages.value.splice(index, 1);
  }
};

const onDragStart = (event, image) => {
  if (selectedImages.value.length === 0) {
    event.dataTransfer.setData('text/plain', JSON.stringify([image]));
  } else if (selectedImages.value.includes(image)) {
    event.dataTransfer.setData('text/plain', JSON.stringify(selectedImages.value));
  }


};

const onDrop = (event) => {
  event.preventDefault();
  console.log(event.dataTransfer.getData('text/plain'))
  const images = JSON.parse(event.dataTransfer.getData('text/plain'));
  images.forEach(image => {
    if (!droppedImages.value.includes(image)) {
      droppedImages.value.push(image);
    }
  });
  // 清空选中状态
  selectedImages.value = [];
};

const startSelection = (event) => {
  console.log('start selection')
  selectionBox.visible = true;
  selectionBox.startX = event.clientX;
  selectionBox.startY = event.clientY;
  updateSelectionBox(event.clientX, event.clientY);
};

const moveSelection = (event) => {
  if (selectionBox.visible) {
    updateSelectionBox(event.clientX, event.clientY);
  }
};

const updateSelectionBox = (mouseX, mouseY) => {
  const left = Math.min(mouseX, selectionBox.startX);
  const top = Math.min(mouseY, selectionBox.startY);
  const width = Math.abs(mouseX - selectionBox.startX);
  const height = Math.abs(mouseY - selectionBox.startY);
  selectionBox.style = {
    left: `${left}px`,
    top: `${top}px`,
    width: `${width}px`,
    height: `${height}px`,
    backgroundColor: `rgba(0, 0, 255, 0.2)`
  };
};

const selectionBox = reactive({
  visible: false,
  startX: 0,
  startY: 0,
  style: {
    left: '0px',
    top: '0px',
    width: '100px',
    height: '100px',
    backgroundColor: 'red',
    zIndex: 9999
  }
});

const endSelection = () => {
  if (selectionBox.visible) {
    selectionBox.visible = false;
    selectedImages.value = []
    selectImagesInBox();
  }
};

const selectImagesInBox = () => {
  // 判断图片是否在选择框内
  const box = selectionBox.style
  const boxLeft = parseInt(box.left.replace('px', ''));
  const boxTop = parseInt(box.top.replace('px', ''));
  const boxWidth = parseInt(box.width.replace('px', ''));
  const boxHeight = parseInt(box.height.replace('px', ''));
  const boxRight = boxLeft + boxWidth
  const boxBottom = boxTop + boxHeight

  images.value.forEach(image => {
    const imageRect = document.querySelector(`img[src='${image}']`).getBoundingClientRect();
    const imageLeft = imageRect.left;
    const imageTop = imageRect.top;
    const imageRight = imageRect.right;
    const imageBottom = imageRect.bottom;

    // 判断图片是否和选择框有交集
    const hasIntersection =
      boxLeft < imageRight &&
      boxRight > imageLeft &&
      boxTop < imageBottom &&
      boxBottom > imageTop;

    if (hasIntersection) {
      selectedImages.value.push(image);
    }
  })
}

const onDragOver = (event) => {
  event.preventDefault();
};

</script>
<template>
  <div class="container">
    <div class="left-panel" @mousedown="startSelection" @mousemove="moveSelection" @mouseup="endSelection">
      <div
        v-for="(image, index) in images"
        :key="index"
        style="width: 200px; height: 200px;"
        :class="['image-item', { selected: selectedImages.includes(image) }]"
        @click="toggleSelect(image)"
        draggable="true"
        @dragstart="(event) => onDragStart(event, image)"
      >
        <img :src="image" alt="image" style="width: 200px; height: 200px;" />
      </div>
      <div
        class="drag-placeholder"
        v-if="selectedImages.length > 0"
        draggable="true"
        @dragstart="onDragStart"
      >
        Drag {{ selectedImages.length }} selected images
      </div>
      <div v-if="selectionBox.visible" class="selection-box" :style="selectionBox.style"></div>
    </div>
    <div class="right-panel" @drop="onDrop" @dragover="onDragOver">
      <div v-for="(image, index) in droppedImages" :key="index" class="dropped-image">
        <img :src="image" alt="dropped image" style="width: 200px; height: 200px;" />
      </div>
    </div>
  </div>
</template>

<style>
.container {
  display: flex;
}
.left-panel, .right-panel {
  flex: 1;
  padding: 10px;
  border: 1px solid #ccc;
  position: relative;
}
.selection-box {
  position: absolute;
}
.image-item {
  cursor: pointer;
  margin-bottom: 10px;
}
.image-item.selected {
  border: 2px solid blue;
}
.dropped-image {
  margin-bottom: 10px;
}
</style>