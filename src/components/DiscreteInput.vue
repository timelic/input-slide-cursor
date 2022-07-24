<script setup lang="ts">
import { HTMLAttributes, ref, withDefaults } from 'vue';

interface Props {
  modelValue: string;
  mountInputCell?: number; // 多少个格子
  width?: number; // 整体宽度
  Height?: number;
  cellWidth?: number; // 如果设置了width，那么cellWidth将不起作用
  cellGap?: number;
  cellStyle?: HTMLAttributes['style'];
  fontSize?: number;
  // 应该有个选项 限制是不是只能输入数字
}

// props
// 这里不能解构 否则会导致丧失ref
const props = withDefaults(defineProps<Props>(), {
  modelValue: '',
  mountInputCell: 6,
  cellGap: 10,
  cellStyle: '',
  height: 30,
});

const emit = defineEmits<{
  (event: 'update:modelValue', modelValue: string): void;
}>();

const inputRefs = new Array(props.mountInputCell).fill(
  ref()
) as HTMLInputElement[];

// 当前格子是空的，输入一个字符之后自动让下一个格子focus
// 当前格子已有字符。focus之后马上清空当前字符。
// 一次性输入n个字符，只会选取第一个文字放入第一个字符

function handleChange(index: number) {
  emit('update:modelValue', inputCellsValue());
  if (index === inputRefs.length - 1 || inputRefs[index + 1].value) return;
  inputRefs[index + 1].focus();
}

function handleFocus(index: number) {
  emit('update:modelValue', inputCellsValue());
  if (index === inputRefs.length - 1) return;
  inputRefs[index].value = '';
}

const inputCellsValue = () =>
  inputRefs.map(elem => elem.value).reduce((prev, current) => prev + current);
</script>

<template>
  <div>
    <template v-for="(_, index) in props.mountInputCell">
      <input
        type="text"
        class="input-cell"
        :ref="el => { inputRefs[index] = el as any }"
        maxlength="1"
        @input="handleChange(index)"
        @focus="handleFocus(index)"
        :style="
          index < props.mountInputCell - 1
            ? cellStyle + `margin-right: ${props.cellGap}px;`
            : cellStyle
        "
      />
    </template>
  </div>
</template>

<style scoped lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Oxanium:wght@500&display=swap');

input {
  border: 1px solid #eee;
  outline: medium;
  background-color: #f1f1f1;
  padding: 0;
  width: 30px;
  height: 30px;
  line-height: 30px;
  border-radius: 5px;
  text-align: center;
  caret-color: #7a7a7a;
  font-size: 17px;
  font-family: 'Oxanium', cursive;
  user-select: none;
  -webkit-user-select: none;
  transition: 0.2s;
  &:hover {
    border: 1px solid rgb(0, 115, 223);
  }
  &:focus {
    background-color: rgb(233, 238, 243);
    border: 1px solid rgb(99, 161, 255);
    box-shadow: 0 0 1px 2px rgb(0 115 223 / 19%);
    animation: inputFocus 0.6s;
    transform: scale(1.05);
  }
}

@keyframes inputFocus {
  25% {
    box-shadow: 0 0 10px 1px rgba(0, 115, 223, 0.3);
  }

  100% {
    box-shadow: 0 0 30px 1px rgba(0, 115, 223, 0),
      0 0 1px 2px rgb(0 115 223 / 19%);
  }
}
</style>
