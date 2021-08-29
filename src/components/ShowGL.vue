<script setup lang="ts">
const props = defineProps<{ title: string }>()
</script>
<template>
  <div class="block lg:flex justify-center gap-4 lg:h-[calc(100vh-70px)] overflow-hidden ">
    <div class="py-6 flex flex-col items-center justify-center">
      <slot name="canvas"></slot>
      <span class="text-gray-500 mt-4">{{ props.title }}</span>
      <GoBack />
    </div>
    <div class="mx-4 box-border flex items-center overflow-y-scroll my-6 overflow-hidden lg:h-[calc(100vh-3rem-70px)]">
      <slot name="readme"></slot>
    </div>
  </div>
</template>
