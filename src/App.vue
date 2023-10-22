<template>
  <main>
    <Editor class="split" :template="getTemplateText()" :vars="vars" />
    <Teleport to="#phone-info" v-if="ready" :disabled="!isPhone">
      <div class="split info">
        <a href="https://github.com/xiao-e-yun/templateCraft">
          <h2>Template Craft
            <svg xmlns="http://www.w3.org/2000/svg" class="github" fill="currentColor" viewBox="0 0 16 16">
              <path
                d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.012 8.012 0 0 0 16 8c0-4.42-3.58-8-8-8z" />
            </svg>
          </h2>
        </a>
        <p>
          Set template, variables.<br>
          Next time change variables then done.
        </p>
        <span><a href="https://xiaoeyun.me">Our Website</a></span>
      </div>
    </Teleport>

  </main>
  <aside>
    <div v-if="vars.length" v-for="(_, i) in vars" :draggable="!!vars[i][0]"
      @dragstart="e => e.dataTransfer!.setData('text', vars[i][0])" class="item">
      <div class="variable-bar">
        <input class="variable-name" type="text" placeholder="Name" v-model="vars[i][0]" @drop.prevent>
        <button @click="vars.splice(i, 1)">X</button>
      </div>
      <input class="variable-value" type="text" placeholder="Value" v-model="vars[i][1]" @drop.prevent>
    </div>
    <div class="no-vars" v-else>
      <h2>Variables</h2>
      <p>Create a vars and show charm</p>
      <p>Drag variable to editor<wbr> or enter "[VAR_NAME]"</p>
    </div>
    <div class="item line" @click="vars.push(['variable', ''])">Create</div>
    <br>
    <div class="item line" @click="exportToJSON">Export Template</div>
    <div class="item line" @click="loadByFile">Load Template</div>
  </aside>

  <div id="phone-info" />
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { useMediaQuery } from "@vueuse/core"
import Editor from './components/Editor.vue';


const ready = ref(false)
onMounted(() => ready.value = true)

const vars = ref<[string, string][]>([])
const templateText = ref("Hello, Use right panel create a [variable].")
function getTemplateText() { return templateText }

const cache = localStorage.getItem("template-craft")
if (cache) loadByJSON(cache)

addEventListener("beforeunload", ()=>localStorage.setItem("template-craft",saveToJSON()))

function exportToJSON() {
  var hiddenElement = document.createElement('a');
  hiddenElement.href = 'data:application/json,' + encodeURI(saveToJSON());
  hiddenElement.target = '_blank';
  hiddenElement.download = 'template.json';
  hiddenElement.click();
}

function saveToJSON() {
  
    return JSON.stringify({
      template: templateText.value,
      vars: vars.value,
    })
}

async function loadByFile() {
  const input = document.createElement("input")
  input.accept = "application/json"
  input.type = "file"
  input.click()
  await new Promise(r=>input.onchange = r)
  const files = input.files
  if(!files || !files.length) return
  const text = await files[0].text()
  loadByJSON(text)
}

function loadByJSON(json: string) {
  try {
    const obj = JSON.parse(json) as {
        template: string,
        vars: [string, string][],
      }
    templateText.value = obj.template
    vars.value = obj.vars
  } catch {}
}

const isPhone = useMediaQuery('(max-width: 720px)')
</script>

<style lang="scss">
@import "./assets/style/page.scss";

#app {
  width: 100vw;
  height: 100vh;
  display: flex;

  overflow: hidden;

  @include pe {
    overflow: auto;
    flex-direction: column;
  }
}

main {
  width: 70%;
  display: flex;
  flex-direction: column;


  @include pe {
    width: 100%;
  }
}

.split {
  height: 50%;
  margin: .4em;
  @include shadow;
  border-radius: .4em;
  background: color.side-3();

  @include pe {
    height: 50vw;
  }
}

aside {
  width: 30%;
  height: 100vh;
  overflow: auto;

  & h2 {
    margin: 1em 0 0 0;
  }

  & p {
    margin: 0;
  }

  & .no-vars {
    @include shadow;
    background: color.side-2();
    padding: 1em 0.4em 1.2em;
    height: auto;

    background: color.side-3();
    border-radius: .4em;
    margin: .4em;

    & h2,
    & p {
      margin: .2em 0;
      text-align: center;
    }
  }

  @include pe {
    height: auto;
    width: calc(100%);
    overflow: visible;
  }

  &>.item {
    margin: .4em;
    @include shadow;
    overflow: hidden;
    border-radius: .4em;
    background: color.side-3();
    transition: background .3s;

    &.line {
      padding: .1em;
      text-align: center;

      &:hover {
        background: color.side-2();
      }
    }

    & .variable-bar {
      width: 100%;
      display: flex;
      justify-content: space-between;
    }

    & .variable-name {
      width: 100%;
      display: block;
      outline: none;
      user-select: none;
    }

    & .variable-value {
      background: color.side-2();
      border-radius: .4em;
      padding: .2em .6em;
      display: block;
      width: 100%;
    }

    & button {
      background: transparent;
    }
  }
}

.info {
  position: relative;
  overflow: overlay;
  padding: 1em;

  & span {
    position: absolute;
    bottom: .5em;
    right: .5em;
  }

  & h2 {
    margin: 0 0 1em 0;
  }
}

.github {
  width: .8em;
  height: .8em;
}
</style>
