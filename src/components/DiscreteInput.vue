<script setup lang="ts">
import { HTMLAttributes, ref, withDefaults, Ref, onMounted, watch } from 'vue';

type acceptType = 'digit' | 'letter' | 'all';

interface InputError {
  code: 100 | 101 | 102;
  message: string;
}

interface Props {
  modelValue: string;
  cellMount?: number; // 多少个格子
  width?: number; // 整体宽度
  height?: number;
  cellWidth?: number; // 如果设置了width，那么cellWidth将不起作用
  cellGap?: number;
  cellStyle?: HTMLAttributes['style'];
  fontSize?: number;
  themeColor?: string;
  acceptType?: acceptType[] | acceptType;
  autoFocus?: boolean;
  onSuccess?: () => void; // 成功输入所有格子的回调
  onError?: (e: InputError) => void; // 输入错误的回调
}

// props
// 这里不能解构 否则会导致丧失ref
const props = withDefaults(defineProps<Props>(), {
  modelValue: '',
  cellMount: 6,
  width: 0,
  height: 40,
  cellWidth: 40,
  cellGap: 10,
  cellStyle: '',
  fontSize: 17,
  themeColor: '#0073df',
  autoFocus: false,
  acceptType: 'digit',
});

// 如果设置了width，那么cellWidth将不起作用
const cellWidth = () =>
  props.width
    ? (props.width - (props.cellMount - 1) * props.cellGap) / props.cellMount
    : props.cellWidth;

const width = () =>
  props.width
    ? props.width
    : props.cellGap * (props.cellMount - 1) + props.cellWidth * props.cellMount;

const emit = defineEmits<{
  (event: 'update:modelValue', modelValue: string): void;
}>();

const fakeInputRefs = new Array(props.cellMount).fill(
  ref()
) as HTMLDivElement[];

let inputRef = ref() as Ref<HTMLInputElement>;

// 当前选中格子索引
let curSelect = ref(-1);

// 所有伪input的值之和
const fakeInputCellsValue = () =>
  fakeInputRefs
    .map(elem => elem.innerText)
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
  // 有这个就很迷 没这个也很迷
  // emit('update:modelValue', fakeInputCellsValue());
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
  // if (!/[0-9]|^$/.test(content)) return;
  content = textFilter(content, props.acceptType);
  if (curSelect.value >= props.cellMount) return;
  fakeInputRefs[curSelect.value].innerText = content;
  moveIndex(step);
}

// 移动当前格索引
function moveIndex(step: number) {
  if (curSelect.value + step >= props.cellMount) return;
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
// 并且强制不能让v-model的值的类型发生错误
watch(
  () => props.modelValue,
  str => {
    emit(
      'update:modelValue',
      textFilter(props.modelValue.slice(0, props.cellMount), props.acceptType)
    );
    fakeInputRefs.forEach((elem, index) => {
      elem.innerText = str.charAt(index);
    });
    if (fakeInputRefs.every(elem => elem.innerText)) {
      props.onSuccess?.();
    }
  }
);

function textFilter(
  str: string,
  acceptType: acceptType | acceptType[]
): string {
  if (typeof acceptType !== 'object') acceptType = [acceptType];
  // 如果是 all 那么直接返回
  if (acceptType.includes('all')) return str;
  let res = str;
  // 如果是两者 那么返回字母和数字
  if (acceptType.includes('digit') && acceptType.includes('letter'))
    res = str.replace(/[^0-9a-zA-Z]/gi, '');
  // 如果是字母
  if (acceptType.length === 1 && acceptType.includes('letter'))
    res = str.replace(/[^a-zA-Z]/gi, '');
  // 如果是数字或者为空
  if (acceptType.length === 1 && acceptType.includes('digit'))
    res = str.replace(/[^0-9]/gi, '');
  if (res.length !== str.length) {
    props.onError?.({
      code: 101,
      message: 'type invalid char.',
    });
  }
  return res;
}

function handleInputError() {
  props.onError?.({
    code: 102,
    message: 'input dom element error.',
  });
}
</script>

<template>
  <div class="discrete-input-container">
    <div class="input-cell-wrap" v-for="(_, index) in props.cellMount">
      <div
        type="text"
        class="fake-input-cell"
        :class="{ 'input-focus': index === curSelect }"
        :ref="el => { fakeInputRefs[index] = el as any }"
        maxlength="1"
        @click="handleFocus(index)"
        :style="cellStyle"
      ></div>
      <span
        class="input-cursor"
        v-if="index === curSelect && !fakeInputRefs[index].innerText"
      ></span>
    </div>
  </div>
  <input
    class="real-input"
    type="number"
    ref="inputRef"
    style="width: 0"
    v-model="inputText"
    @input="handleInputChange"
    @keyup.delete.native="handleDelete"
    @blur="handleBlur"
    @error="handleInputError"
  />
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Oxanium:wght@500&display=swap');
</style>

<style scoped lang="scss">
$theme-color: #0073df;
// $theme-color: v-bind('props.themeColor');
// $theme-color: red;

input {
  outline: medium;
  border: none;
  padding: 0;
  box-sizing: border-box;
}

.discrete-input-container {
  display: flex;
  width: calc(v-bind(width()) * 1px);
  justify-content: space-between;
  --theme-color: #0073df;
}

.input-cell-wrap {
  position: relative;
  width: calc(v-bind(cellWidth()) * 1px);
  font-size: calc(v-bind('props.fontSize') * 1px);
}

.fake-input-cell {
  border: 1px solid #eee;
  outline: medium;
  background-color: #f1f1f1;
  padding: 0;
  width: calc(v-bind(cellWidth()) * 1px);
  height: calc(v-bind('props.height') * 1px);
  line-height: calc(v-bind('props.height') * 1px);
  border-radius: 5px;
  text-align: center;
  caret-color: #7a7a7a;
  color: black;
  font-family: 'Oxanium', cursive;
  user-select: none;
  -webkit-user-select: none;
  cursor: text;
  transition: 0.2s;
  &:hover {
    border: 1px solid $theme-color;
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
  width: 0.65em;
  height: 0.15em;
  background-color: #9b9b9b;
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
