<script setup>
import DefaultTheme from 'vitepress/theme'
import { onMounted, watch, nextTick } from 'vue'
import { useRouter } from 'vitepress'
import mediumZoom from 'medium-zoom'

const { Layout } = DefaultTheme
const router = useRouter()
let zoom = null

const setupZoom = () => {
  nextTick(() => {
    if (zoom) zoom.detach()
    zoom = mediumZoom('.vp-doc img', {
      background: 'transparent'
    })
  })
}

onMounted(setupZoom)
watch(() => router.route.path, () => {
  setupZoom()
})
</script>

<template>
  <Layout />
</template>

<style>
.medium-zoom-overlay {
  backdrop-filter: blur(5px);
}

.medium-zoom-overlay,
.medium-zoom-image--opened {
  z-index: 9999;
}
</style>
