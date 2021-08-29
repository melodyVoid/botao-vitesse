<script setup lang="ts">
import { isDark, toggleDark } from '~/logic'
const { t } = useI18n()
</script>
<template>
  <div class="py-5 px-7 flex items-center justify-end text-gray-700 dark:text-gray-200">
    <button class="icon-btn mx-2 !outline-none" text="lg" :title="t('button.toggle_dark')" @click="toggleDark()">
      <carbon-moon v-if="isDark" />
      <carbon-sun v-else />
    </button>
  </div>
</template>
