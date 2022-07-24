<script setup lang="ts">
import { HTMLAttributes, ref, withDefaults, Ref, onMounted } from 'vue';

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

// const emit = defineEmits<{
//   (event: 'update:modelValue', modelValue: string): void;
// }>();

const inputRefs = new Array(props.mountInputCell).fill(
  ref()
) as HTMLInputElement[];

let inputRef = ref() as Ref<HTMLInputElement>;

// 当前选中格子索引
let curSelect = ref(0) as Ref<number>;

// 当前格子是空的，输入一个字符之后自动让下一个格子focus
// 当前格子已有字符。focus之后马上清空当前字符。
// 一次性输入n个字符，只会选取第一个文字放入第一个字符

// function handleChange(index: number) {
//   emit('update:modelValue', inputCellsValue());
//   if (index === inputRefs.length - 1 || inputRefs[index + 1].value) return;
//   inputRefs[index + 1].focus();
// }

function handleFocus(index: number) {
  inputRef.value.focus()
  curSelect.value = index
  // emit('update:modelValue', inputCellsValue());
  // if (index === inputRefs.length - 1) return;
  // inputRefs[index].value = '';
}

function handleDelete() {
  replaceContent('', -1)
}

// 防止中文输入法导致多次输入
let inputTimer = null as number | null;
function handleInputChange(e: InputEvent) {
  if (inputTimer) return
  replaceContent(e.data, 1)
  inputText.value = ''
  inputTimer = setTimeout(function() {
    inputTimer = null
  }, 0)
}

// 取代当前格内容
function replaceContent(content: (string | null | undefined), step: number) {
  if (content === undefined || content === null) return
  if (!/[0-9]|^$/.test(content)) return
  if (curSelect.value < 0 || curSelect.value >= props.mountInputCell) return
  inputRefs[curSelect.value].value = content
  moveIndex(step)
}

// 移动当前格索引
function moveIndex(step: number) {
  if (curSelect.value + step < 0 || curSelect.value + step >= props.mountInputCell) return
  curSelect.value += step
}

// const inputCellsValue = () =>
//   inputRefs.map(elem => elem.value).reduce((prev, current) => prev + current);

const inputText = ref('');

onMounted(() => {
  inputRef.value.focus()
})
</script>

<template>
  <div>
    <template v-for="(_, index) in props.mountInputCell">
      <input
        type="text"
        class="input-cell"
        :class='{ "input-focus": index === curSelect }'
        :ref="el => { inputRefs[index] = el as any }"
        maxlength="1"
        @focus="handleFocus(index)"
        :style="
          index < props.mountInputCell - 1
            ? cellStyle + `margin-right: ${props.cellGap}px;`
            : cellStyle
        "
      />
    </template>
    <input ref='inputRef' v-model='inputText' @input='handleInputChange' @keyup.delete.native='handleDelete' style='width: 0'/>
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
</style>
