<script lang="ts" setup>
import gsap from 'gsap'
const props = withDefaults(
  defineProps<{
    content?: string
  }>(),
  {
    content: ''
  }
)
const text = computed(() => [...props.content])
const spans: Ref<HTMLSpanElement[]> = ref([])
onMounted(async () => {
  gsap.from(spans.value, {
    duration: 2,
    stagger: 0.05,
    x: 30,
    y: 25,
    opacity: 0,
    ease: 'back.inOut(5)'
  })
})
</script>

<template>
  <span class="inline-block" v-for="i in text" ref="spans">{{ i }}</span>
</template>
