<template>
  <div :class="$style.editor" @dragover.prevent @drop.prevent="onDrop">
    <textarea v-model="props.template.value" @input="changeHeight" :style="{ '--height': height + 'px' }"
      @focusin="focus = true" @blur="focus = false" />
    <pre v-html="preview" ref="previewEl" :class="{[$style.focus]: focus}"></pre>
    <button @click="copy" :class="$style.copy">Copy</button>
  </div>
</template>

<script lang="ts" setup>
import { computed, ref, Ref } from 'vue';

const focus = ref(false)

const previewEl = ref<HTMLPreElement>()
function changeHeight() { height.value = previewEl.value!.clientHeight }
const height = ref(24)

const props = defineProps<{
  template: Ref<string>
  vars: [string, string][]
}>()

const parseHTML = (() => {
  const dom = document.createElement('div');
  return (text: string) => {
    dom.innerText = text;
    return dom.innerHTML;
  }
})();

function onDrop(e: DragEvent) {
  const text = e!.dataTransfer!.getData("text")
  if (text) props.template.value += ` [${text}]`
}

async function copy() {
  const vars = props.vars.map(v=>v[0])
  const text = template.value.map(template => {
    const isVarFormat = template.length && template.startsWith("[") && template.endsWith("]")
    const replacedVar = isVarFormat ? template.replace(/\[(.*)\]/, "$1") : ""
    const isVar = isVarFormat && vars.includes(replacedVar)
    const varIndex = isVar ? vars.indexOf(replacedVar) : 0
    const varValue = props.vars[varIndex][1]
    return isVar ? varValue : template
  }).join(" ").replaceAll(" \n ","\n")
  await navigator.clipboard.writeText(text)
}


const template = computed(() => props.template.value.replaceAll("\n", " \n ").split(" "))
const preview = computed(() => {
  const vars = props.vars.map(v=>v[0])
  return template.value.map(template => {
    const isVarFormat = template.length && template.startsWith("[") && template.endsWith("]")
    const replacedVar = isVarFormat ? template.replace(/\[(.*)\]/, "$1") : ""
    const isVar = isVarFormat && vars.includes(replacedVar)
    const text = parseHTML(template)
    const varIndex = isVar ? vars.indexOf(replacedVar) : 0
    const varValue = isVar && (focus.value ? replacedVar : props.vars[varIndex][1])
    return isVar ? `<span>${text}<span title=${varValue}>${varValue}</span></span>` : text
  }).join(" ").replaceAll(" <br> ","<br>")
})
</script>

<style lang="scss" module>
.editor {
  position: relative;
  color: transparent;
  overflow: overlay;
  display: flex;

  .copy {
    position: absolute;
    background: color.main();
    color: color.bg();
    bottom: .4em;
    right: .4em;

    &:hover {
      background: color.side-1();
    }
  }

  pre {
    top: 0;
    margin: 0;
    z-index: 1;
    padding: .4em;
    text-wrap: pretty;
    color: transparent;
    position: absolute;
    user-select: none;
    pointer-events: none;
    
    &.focus span {
      pointer-events: none;
    }

    &>span {
      color: transparent;
      position: relative;
      background: color.side-3();

      &>span {
        top: 0;
        left: 0;
        overflow: hidden;
        user-select: all;
        pointer-events: all;
        text-overflow: ellipsis;
        width: calc(100% - .2rem);
        height: calc(100% - .2rem);
        margin: .1rem;
        font-size: .8em;
        font-weight: bold;
        position: absolute;
        color: color.bg();
        background: color.main();
        border-radius: .2em;
        text-align: center;

      }
    }
  }

  & textarea {
    padding: 0;
    width: 100%;
    resize: none;
    padding: .4em;
    display: block;
    font-size: 1em;
    min-height: 100%;
    height: var(--height);
    background: transparent;
  }
}
</style>