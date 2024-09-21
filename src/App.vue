<script setup lang="ts">
import type { ComputedRef, Ref } from 'vue'
import { useIntervalFn } from '@vueuse/core'
import { computed, reactive, ref } from 'vue'

const images: Ref<string[]> = ref([])

const currentIndex = ref(0)
// トランジションの実装
const imagesToShow = reactive([
  { url: images.value[0], opacity: 1 },
  { url: images.value[1], opacity: 0 },
])

useIntervalFn(() => {
  const nextIndex = (currentIndex.value + 1) % images.value.length

  // 次の画像を準備
  imagesToShow.push({ url: images.value[nextIndex], opacity: 0 })

  // フェードイン・フェードアウトを開始
  imagesToShow[0].opacity = 0
  imagesToShow[1].opacity = 1

  // トランジションが終わったら古い画像を削除
  setTimeout(() => {
    imagesToShow.shift()
  }, 1000)

  currentIndex.value = nextIndex
}, 5000)

function onSelected(event: Event) {
  const inputElement = event.target as HTMLInputElement
  if (!inputElement.files) {
    return
  }
  const files: FileList = inputElement.files
  const imageFiles: File[] = Array.from(files).filter((file: File) => file.type.startsWith('image/'))
  imageFiles.forEach(v => images.value.push(URL.createObjectURL(v)))
}
</script>

<template>
  {{ imagesToShow }}
  <div
    v-for="(image, index) in imagesToShow"
    :key="index"
    class="absolute inset-0 bg-cover bg-center transition-opacity duration-1000"
    :style="{
      backgroundImage: `url(${image.url})`,
      opacity: image.opacity,
    }"
  />
  <input id="folderInput" class="z-100 absolute" type="file" webkitdirectory directory multiple @input="onSelected">
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 1s ease-in-out;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
