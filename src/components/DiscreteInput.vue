<script setup lang="ts">
import { HTMLAttributes, ref, withDefaults, Ref, onMounted, watch } from 'vue';

type acceptType = 'digit' | 'letter' | 'all';

interface Props {
  modelValue: string;
  mountInputCell?: number; // 多少个格子
  width?: number; // 整体宽度
  Height?: number;
  cellWidth?: number; // 如果设置了width，那么cellWidth将不起作用
  cellGap?: number;
  cellStyle?: HTMLAttributes['style'];
  fontSize?: number;
  acceptType?: acceptType[] | acceptType;
  autoFocus?: boolean;
  onSuccess?: () => void; // 成功输入所有格子的回调
  onError?: () => void; // 输入错误的回调
}

// props
// 这里不能解构 否则会导致丧失ref
const props = withDefaults(defineProps<Props>(), {
  modelValue: '',
  mountInputCell: 6,
  cellGap: 10,
  cellStyle: '',
  height: 30,
  autoFocus: false,
});

const emit = defineEmits<{
  (event: 'update:modelValue', modelValue: string): void;
}>();

const fakeInputRefs = new Array(props.mountInputCell).fill(
  ref()
) as HTMLInputElement[];

let inputRef = ref() as Ref<HTMLInputElement>;

// 当前选中格子索引
let curSelect = ref(-1);

// 所有伪input的值之和
const fakeInputCellsValue = () =>
  fakeInputRefs
    .map(elem => elem.value)
    .reduce((prev, current) => prev + current);

// real-input的v-model的值
const inputText = ref('');

// 当前格子是空的，输入一个字符之后自动让下一个格子focus
// 当前格子已有字符。focus之后马上清空当前字符。
// 一次性输入n个字符，只会选取第一个文字放入第一个字符

function handleFocus(index: number) {
  inputRef.value.focus();
  curSelect.value = index;
}

function handleDelete() {
  replaceContent('', -1);
}

// 防止中文输入法导致多次输入
let inputTimer = null as number | null;
function handleInputChange(e: Event) {
  if (inputTimer) return;
  replaceContent((e as InputEvent).data, 1);
  inputText.value = '';
  inputTimer = setTimeout(function () {
    inputTimer = null;
  }, 0);
  emit('update:modelValue', fakeInputCellsValue());
}

// 取代当前格内容
function replaceContent(content: string | null | undefined, step: number) {
  if (content === undefined || content === null) return;
  if (!/[0-9]|^$/.test(content)) return;
  if (curSelect.value >= props.mountInputCell) return;
  fakeInputRefs[curSelect.value].value = content;
  moveIndex(step);
}

// 移动当前格索引
function moveIndex(step: number) {
  if (curSelect.value + step >= props.mountInputCell) return;
  curSelect.value += step;
}

// 是否自动聚焦
onMounted(() => {
  if (props.autoFocus) inputRef.value.focus();
});

// 取消聚焦
function handleBlur() {
  curSelect.value = -1;
}

// 当v-model的值被外界改变时候，自动更新fake-input的状态
watch(
  () => props.modelValue,
  str => {
    fakeInputRefs.forEach((elem, index) => {
      elem.value = str.charAt(index);
    });
    if (fakeInputRefs.every(elem => elem.value)) {
      props.onSuccess?.();
    }
  }
);
</script>

<template>
  <div class="discrete-input-container">
    <div
      class="input-cell-wrap"
      :style="
        index < props.mountInputCell - 1
          ? cellStyle + `margin-right: ${props.cellGap}px;`
          : cellStyle
      "
      v-for="(_, index) in props.mountInputCell"
    >
      <input
        type="text"
        class="fake-input-cell"
        :class="{ 'input-focus': index === curSelect }"
        :ref="el => { fakeInputRefs[index] = el as any }"
        maxlength="1"
        @focus="handleFocus(index)"
        :style="cellStyle"
      />
      <span
        class="input-cursor"
        v-if="index === curSelect && !fakeInputRefs[index].value"
      ></span>
    </div>
    <input
      class="real-input"
      type="number"
      ref="inputRef"
      v-model="inputText"
      @input="handleInputChange"
      @keyup.delete.native="handleDelete"
      style="width: 0"
      @blur="handleBlur"
    />
  </div>
</template>

<style scoped lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Oxanium:wght@500&display=swap');

input {
  outline: medium;
  border: none;
  padding: 0;
}

.discrete-input-container {
  display: flex;
}

.input-cell-wrap {
  position: relative;
}

.fake-input-cell {
  border: 1px solid #eee;
  outline: medium;
  background-color: #f1f1f1;
  padding: 0;
  width: 40px;
  height: 40px;
  line-height: 40px;
  border-radius: 5px;
  text-align: center;
  caret-color: #7a7a7a;
  font-size: 17px;
  font-family: 'Oxanium', cursive;
  user-select: none;
  -webkit-user-select: none;
  transition: 0.2s;
  padding-top: 10%;
  &:hover {
    border: 1px solid rgb(0, 115, 223);
  }
  &.input-focus {
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

.input-cursor {
  width: 30%;
  height: 7%;
  background-color: rgb(155, 155, 155);
  display: block;
  position: absolute;
  top: 70%;
  left: 50%;
  transform: translateX(-50%);
  animation: fadeIn 0.6s alternate-reverse infinite linear;
}

@keyframes fadeIn {
  0% {
    opacity: 0;
  }
  70% {
    opacity: 1;
  }
}
</style>
