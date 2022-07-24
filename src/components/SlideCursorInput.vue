<script setup lang="ts">
import { computed, ref, withDefaults, HTMLAttributes, watch, Ref } from 'vue';

interface Props {
  modelValue: string;
  style?: HTMLAttributes['style'];
}

// props
// 这里不能解构 否则会导致丧失ref
const props = withDefaults(defineProps<Props>(), { modelValue: '' });

// emit
const emit = defineEmits<{
  (event: 'update:modelValue', modelValue: string): void;
}>();

let initial = '';

const inputText = computed({
  get: () => props.modelValue,
  set: (value) => {
    emit('update:modelValue', value);
    if (!isTextAnimating.value) checkInput(value);
  },
});

const inputElementRef = ref() as Ref<HTMLInputElement>;

const isTextAnimating = ref(false);

let [originSelectionStart, originSelectionEnd] = [0, 0];

function checkInput(target: string) {
  if (!isTextAnimating.value) {
    console.log({ initial, target });
    isTextAnimating.value = true;
    inputText.value = '';
    // inputText.value = initial;
    // 光标位置
    inputElementRef.value.selectionStart;
    const [currentSelectionStart, currentSelectionEnd] = [
      inputElementRef.value.selectionStart!,
      inputElementRef.value.selectionEnd!,
    ];
    console.log({ currentSelectionStart, currentSelectionEnd });
    // if (originSelectionStart < currentSelectionEnd) {
    //   // 原来的左光标 < 新的左光标，意味着输入了文字
    //   let pos = originSelectionStart;
    //   let timerId = setInterval(() => {
    //     inputText.value = inputText.value + target.charAt(pos);
    //     pos++;
    //     if (pos <= currentSelectionEnd) clearInterval(timerId);
    //   }, 200);
    // }
    // isTextAnimating.value = false;
  }
}

// 一些变量
const isFocusing = ref(false);
function focusInput() {
  isFocusing.value = !isFocusing.value;
}

const temporaryStoreText = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value),
});

// 这么想是不对的 应该有一个动画队列

function handleInput(e: Event) {
  console.warn(e);
  // emit('update:modelValue', );
}
</script>

<template>
  <div
    class="input-wrapper"
    :style="props.style"
    :class="isFocusing ? 'focus' : ''"
  >
    <input
      type="text"
      :value="inputText"
      @input="handleInput"
      placeholder="滑动光标输入框"
      @focusin="focusInput"
      @focusout="focusInput"
      ref="inputElementRef"
    />
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
  transition: 0.2s;

  &:hover {
    border: 1px solid rgb(0, 115, 223);
  }
}

.focus {
  border: 1px solid rgb(0, 115, 223);
  box-shadow: 0 0 1px 2px rgb(0 115 223 / 19%);
  animation: inputFocus 0.6s;

  ::-webkit-input-placeholder {
    opacity: 0.8;
  }
}

@keyframes inputFocus {
  25% {
    box-shadow: 0 0 10px 1px rgba(0, 115, 223, 0.2);
  }

  100% {
    box-shadow: 0 0 30px 1px rgba(0, 115, 223, 0),
      0 0 1px 2px rgb(0 115 223 / 19%);
  }
}
</style>
