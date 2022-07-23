<script setup lang="ts">
import { computed, ref, withDefaults, HTMLAttributes } from 'vue'

interface Props {
  modelValue: string
  style?: HTMLAttributes['style']
}

// props
// 这里不能解构 否则会导致丧失ref
const props = withDefaults(defineProps<Props>(), { modelValue: '' });

// emit
const emit = defineEmits<{
  (event: 'update:modelValue', modelValue: string): void
}>();

const inputText = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
})

// 一些变量
const isFocusing = ref(false)
function focusInput() {
  isFocusing.value = !isFocusing.value
}
</script>

<template>
  <div class="input-wrapper" :style="props.style" :class="isFocusing ? 'focus' : ''">
    <input type="text" v-model="inputText" placeholder="滑动光标输入框" @focusin="focusInput" @focusout="focusInput">
    <div class="slide-cursor"></div>
  </div>
</template>

<style scoped lang="scss">
// TODO 将颜色的变量抽出来放到一个地方管理，颜色也需要微调
input {
  border: none;
  outline: medium;
  height: 100%;
  padding: 0;
  text-indent: 0.5rem;
  width: 100%;
  caret-color: rgb(0, 127, 201);

  &::selection {
    background-color: rgb(47, 231, 255);
  }
}

.input-wrapper {
  height: 2rem;
  border: 1px solid gainsboro;
  display: inline-block;
  border-radius: 3px;
  transition: .2s;
}

.focus {
  border: 1px solid rgb(0, 115, 223);
  box-shadow: 0 0 4px 1px rgb(0 115 223 / 19%);
}
</style>
