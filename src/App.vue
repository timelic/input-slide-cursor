<script setup lang="ts">
import { computed, ref, watch, HTMLAttributes } from "vue";
import SlideCursorInput from "./components/SlideCursorInput.vue"

const inputText = ref('')
watch(inputText, () => {
  console.warn(inputText.value)
})


const slideCursorInputStyle: HTMLAttributes['style'] = {
  margin: '20px',
  width: '250px',
}
</script>

<template>
  <img src="/vite.svg" class="logo" alt="Vite logo" />

  <div>
    <SlideCursorInput v-model="inputText" :style="slideCursorInputStyle" />
  </div>

  <input type="text" v-model="inputText" placeholder="普通输入框">
  <div></div>
</template>

<style scoped lang="scss">
div,
span,
p {
  box-sizing: border-box;
}

.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;

  &:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
}
</style>
