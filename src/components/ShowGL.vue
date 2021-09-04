<script setup lang="ts">
const props = defineProps<{ title: string; link?: string }>()
</script>
<template>
  <div class="block lg:flex justify-center gap-4 lg:h-[calc(100vh-70px)] overflow-hidden ">
    <div class="py-6 flex flex-col items-center justify-center">
      <slot name="canvas"></slot>
      <div class="flex items-center mt-4">
        <span class="text-gray-500 mr-3">{{ props.title }}</span>
        <a v-if="props.link" :href="props.link" class="flex items-center" target="_blank" title="Edit on CodeSandbox">
          <simple-icons:codesandbox class="text-gray-500 cursor-pointer" />
        </a>
      </div>
      <GoBack />
    </div>
    <div class="mx-4 box-border flex items-center overflow-y-scroll my-6 overflow-hidden lg:h-[calc(100vh-3rem-70px)]">
      <slot name="readme"></slot>
    </div>
  </div>
</template>
