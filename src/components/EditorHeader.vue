<script setup>
import { ref, onMounted, onUnmounted } from "vue";
import _ from "lodash";

const editor = ref(null);
const undoOrRedo = ref(false);

const history = { forward: [], back: [] };

function onUndo() {
  undoOrRedo.value = true;
  if (!history.back.length) {
    return;
  }
  history.forward.push(editor.value.innerHTML);
  editor.value.innerHTML = history.back.pop();
}
function onRedo() {
  undoOrRedo.value = true;
  if (!history.forward.length) {
    return;
  }

  history.back.push(editor.value.innerHTML);
  editor.value.innerHTML = history.forward.pop();
}
function onIncrease() {
  const selection = window.getSelection();
  document.execCommand("insertHTML", false, `<h2>${selection.toString()}</h2>`);
}
function onDecrease() {
  document.execCommand("formatBlock", false, "p");
}
function insertImage() {
  undoOrRedo.value = true;
  onChange();
  const url = prompt("Напишите URL-адрес изображения");
  document.execCommand("insertHTML", false, `<img src="${url}" alt="image" />`);
}
async function copyHtml() {
  try {
    await navigator.clipboard.writeText(editor.value.innerHTML);
  } catch (err) {
    console.error("Failed to copy: ", err);
  }
}

const onChange = _.debounce(() => {
  if (undoOrRedo.value) {
    undoOrRedo.value = false;
    return;
  }

  if (
    !history.back.length ||
    history.back[history.back.length - 1] != editor.value.innerHTML
  ) {
    history.back.push(editor.value.innerHTML);
  }
}, 200);

onMounted(() => {
  editor.value = document.getElementById("editor");
  editor.value.addEventListener("DOMSubtreeModified", onChange);
});

onUnmounted(() => {
  editor.value.removeEventListener("DOMSubtreeModified", onChange);
});
</script>

<template>
  <div class="header">
    <button class="header__btn" @click="onUndo">
      <img src="@/assets/icons/back.svg" />
    </button>
    <button class="header__btn" @click="onRedo">
      <img src="@/assets/icons/forward.svg" />
    </button>
    <button class="header__btn" @click="onIncrease">
      <img src="@/assets/icons/increase.svg" />
    </button>
    <button class="header__btn" @click="onDecrease">
      <img src="@/assets/icons/decrease.svg" />
    </button>
    <button class="header__btn" @click="insertImage">
      <img src="@/assets/icons/image.svg" />
    </button>
    <button @click="copyHtml" class="header__btn header__btn--copying">
      Скопировать HTML
    </button>
  </div>
</template>

<style>
.header {
  display: flex;
  align-items: center;
  column-gap: 12px;
}
.header__btn {
  background: #282828;
  border-radius: 4px;
  min-width: 42px;
  height: 38px;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
}

.header__btn--copying {
  font-family: "Roboto";
  font-weight: 400;
  font-size: 15px;
  line-height: 23px;
  color: #639eff;
  background: inherit;
}
</style>
