<script lang="ts" setup>
import MerText from '@/components/MerText.vue'
import gsap from 'gsap'
type file = File & {
  url: string
  file: File
  typeEasy: string
  toggetPlay?: (...arg: any[]) => void
  isPlay: boolean
}
const files: file[] = reactive([])
// 素材管理
function uploadFiles(e: Event) {
  const uploadFiles = (e.target as HTMLInputElement).files ?? []
  for (const file of Object.values(uploadFiles)) {
    if (['image', 'video', 'audio'].includes(file.type.split('/')[0])) {
      const url = useObjectUrl(file).value!
      if (url) {
        // @ts-ignore
        let temp: file = {
          arrayBuffer: file.arrayBuffer,
          lastModified: file.lastModified,
          name: file.name,
          size: file.size,
          type: file.type,
          webkitRelativePath: file.webkitRelativePath,
          slice: file.slice,
          stream: file.stream,
          text: file.text,
          file: file,
          isPlay: false,
          url,
          typeEasy: file.type.split('/')[0],
        }
        if (temp.typeEasy === 'video')
          temp.toggetPlay = function (e: HTMLVideoElement) {
            if (this.isPlay) {
              e.pause()
            } else {
              e.play()
            }
            this.isPlay = !this.isPlay
          }
        files.push(temp)
      }
    }
  }
}
function cleanFiles(e: MouseEvent) {
  ;(e.target as HTMLInputElement).value = ''
}
const audioContaier: Ref<HTMLElement[]> = ref([])
function delFile(src: string) {
  files.splice(
    files.findIndex((e) => e.url === src),
    1
  )
}
// 文件上传框动画
const inputFile: Ref<HTMLElement | null> = ref(null)
onMounted(() => {
  gsap.fromTo(
    inputFile.value,
    {
      width: 0,
      opacity: 0,
      overflow: 'hidden'
    },
    {
      ease: 'power1.inOut',
      delay: 0.5,
      duration: 1.5,
      width: '100%',
      opacity: 1,
      overflow: 'visible'
    }
  )
})
let element: HTMLElement | null = null
function dragstart(e: DragEvent) {
  if (e.dataTransfer) {
    e.dataTransfer.dropEffect = 'move'
  }
  element = e.target as HTMLElement
}
function drop(e: DragEvent) {
  if (element) {
    ;(e.currentTarget as HTMLElement).append(element)
    element = null
  }
}
</script>

<template>
  <main class="grid h-full grid-cols-5 grid-rows-2 gap-2 p-2">
    <!-- 属性设置 -->
    <section class="col-span-2 row-span-1" @dragover.prevent @drop.prevent="drop">
      <header><mer-text content="属性设置" /></header>
    </section>
    <!-- 预览窗口 -->
    <section class="col-span-2 row-span-1" @dragover.prevent @drop.prevent="drop">
      <header><mer-text content="预览窗口" /></header>
    </section>
    <!-- 效果面板 -->
    <section class="col-span-1 row-span-2" @dragover.prevent @drop.prevent="drop">
      <header><mer-text content="效果面板" /></header>
    </section>
    <!-- 素材管理 -->
    <section class="col-span-1 row-span-1" >
      <header><mer-text content="素材管理" /></header>
      <div ref="inputFile">
        <input
          type="file"
          accept="image/*,video/*,audio/*"
          multiple
          @change="uploadFiles"
          @click="cleanFiles"
          class="block w-full rounded-lg border border-gray-200 bg-white px-3 py-2 text-sm text-gray-600 placeholder-gray-400/70 file:rounded-full file:border-none file:bg-gray-200 file:px-4 file:py-1 file:text-sm file:text-gray-700 focus:border-blue-400 focus:outline-none focus:ring focus:ring-blue-300 focus:ring-opacity-40 dark:border-gray-600 dark:bg-gray-900 dark:text-gray-300 dark:placeholder-gray-500 dark:file:bg-gray-800 dark:file:text-gray-200 dark:focus:border-blue-300"
        />
      </div>
      <div
        class="grid grid-cols-[repeat(auto-fit,minmax(80px,1fr))] gap-4 overflow-auto rounded-lg"
        @dragover.prevent @drop.prevent="drop"
      >
        <div
          v-for="file in files"
          :key="file.url"
          class="flex flex-col"
          :title="file.name"
          draggable="true"
          @dragstart="dragstart"
        >
          <img
            draggable="false"
            v-if="file.typeEasy === 'image'"
            :src="file.url"
            :alt="file.name"
            class="w-full flex-1 rounded-lg object-cover"
          />
          <video
            v-if="file.typeEasy === 'video'"
            :src="file.url"
            :alt="file.name"
            preload="metadata"
            class="w-full flex-1 rounded-lg object-cover"
            @dblclick="file.toggetPlay?.($event.target)"
          ></video>
          <div
            v-if="file.typeEasy === 'audio'"
            class="flex w-full flex-1 flex-col justify-center rounded-lg"
            :data-src="file.url"
            ref="audioContaier"
            @dblclick="file.toggetPlay?.()"
          ></div>
          <div class="flex justify-evenly">
            <div class="overflow-hidden text-ellipsis whitespace-nowrap">
              <mer-text :content="file.name" />
            </div>
            <button @click="delFile(file.url)" class="whitespace-nowrap text-blue-400">
              <mer-text content="删除" />
            </button>
          </div>
        </div>
      </div>
    </section>
    <!-- 时间线 -->
    <section class="col-span-3 row-span-1" @dragover.prevent @drop.prevent="drop">
      <header><mer-text content="时间线" /></header>
    </section>
  </main>
</template>

<style scoped >
section {
  @apply flex flex-col gap-4 overflow-auto rounded-lg border border-gray-200 p-4 dark:border-none dark:bg-gray-800;
}

header {
  @apply text-center;
}
</style>
