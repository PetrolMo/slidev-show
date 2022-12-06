<script setup lang="ts">
import { ref, Ref, computed, watch } from 'vue'

enum FilterMaps {
  Gray = 0,
  Blur = 1,
  Brightness = 2,
  Contrast = 3,
  Opacity = 4,
  Sepia = 5,
  Saturate = 6,
  DropShadow = 7
}
interface FilterItem {
  key: string,
  value: string | number | undefined,
  default: string |number,
  min: string | number,
  max: string | number,
  unit: string
}
interface Filter {
  [key: FilterMaps]: FilterItem
}
const defaultList = {
  0: {
    key: 'grayscale',
    unit: '%',
    min: 0,
    max: 100,
    default: 0,
    value: 0
  },
  1: {
    key: 'blur',
    unit: 'px',
    min: 0,
    max: 10,
    default: 0,
    value: undefined
  },
  2: {
    key: 'brightness',
    unit: '%',
    min: 0,
    max: 200,
    default: 100,
    value: undefined
  },
  3: {
    key: 'contrast',
    unit: '%',
    min: 0,
    max: 200,
    default: 100,
    value: undefined
  },
  4: {
    key: 'opacity',
    unit: '%',
    min: 0,
    max: 100,
    default: 100,
    value: undefined
  },
  5: {
    key: 'sepia',
    unit: '%',
    min: 0,
    max: 100,
    default: 0,
    value: undefined
  },
  6: {
    key: 'saturate',
    unit: '%',
    min: 0,
    max: 200,
    default: 100,
    value: undefined
  },
  7: {
    key: 'drop-shadow',
    unit: {
      offsetX: 'px',
      offsetY: 'px',
      blurRadius: 'px',
      color: ''
    },
    default: [0, 0, 5, '#000000'],
    value: [0, 0, 5, '#000000'],
    selected: false
  }
}
const props = defineProps({
  showComponent: {
    type: Boolean,
    default: true
  },
  css: {
    type: String,
    default: ''
  },
  width: {
    type: [Number, String]
  },
  height: {
    type: [Number, String]
  },
  unShowCss: {
    type: Boolean,
    default: false
  }
})
let isMultiple = ref(false)
watch(isMultiple, (val) => {
  if(!val) {
    filterList.value = JSON.parse(JSON.stringify(defaultList))
  }
})
let cssLabel:Ref<string> = ref(props.css)
const filterList: Ref<Filter> = ref(JSON.parse(JSON.stringify(defaultList)))
const selected: Ref<FilterMaps> = ref(FilterMaps.Blur)
let sliderValue: Ref<number> = ref(0)
function paintImage() {
  if (!isMultiple.value) {
    cssLabel.value = `${filterList.value[selected.value].key}(${filterList.value[selected.value].value}${filterList.value[selected.value].unit})`
  }
}
watch(sliderValue, (val: number) => {
  const item = filterList.value[selected.value]
  filterList.value[selected.value].value = ((item.max - item.min) / 100 * val).toFixed(0)
  paintImage()
})
watch(selected, (val: FilterMaps) => {
  const value = filterList.value[val].value ? filterList.value[val].value : filterList.value[val].default
  if (val === FilterMaps.DropShadow) {
    filterList.value[val].selected = !filterList.value[val].selected
    cssLabel.value = `drop-shadow(${value[0]}px ${value[1]}px ${value[2]}px ${value[3]})`
    return
  }
  sliderValue.value = Number((value / filterList.value[val].max * 100).toFixed(0))
})
watch(filterList.value[FilterMaps.DropShadow].value, (val) => {
  cssLabel.value = `drop-shadow(${val[0]}px ${val[1]}px ${val[2]}px ${val[3]})`
})
const currentLabel = computed(() => {
  const value = filterList.value[selected.value].value ? filterList.value[selected.value].value : filterList.value[selected.value].default
  return value +  filterList.value[selected.value].unit
})
const selectedList = computed(() => {
  const arr: Array<number> = [0]
  for (const key in filterList.value as FilterMaps) {
    if (filterList.value[key].value) {
      arr.push(Number(key))
    }
  }
  return Array.from(new Set(arr))
})
watch(selectedList, (val, oldVal) => {
  if (!isMultiple.value) return
  cssLabel.value = ''
  selectedList.value.forEach((key: FilterMaps) => {
    if (key === FilterMaps.DropShadow) {
      const val = filterList.value[key].value
      cssLabel.value += ` drop-shadow(${val[0]}px ${val[1]}px ${val[2]}px ${val[3]})`
    }
    else {
      cssLabel.value +=  ` ${filterList.value[key].key}(${filterList.value[key].value}${filterList.value[key].unit})`
    }
  })
})
</script>

<template>
  <div class="main mt-2" v-if="props.showComponent">
    <div class="container">
      <img class="image" src="/exm.jpg" alt="" :style="{ filter: cssLabel, width: `${props.width}px` }">
      <div class="mt-2">
        <input id="multiple" type="checkbox" v-model="isMultiple">
        <label for="multiple" class="ml-1">混合使用</label>
      </div>
      <div class="mt-2">
        <select id="filter" v-model="selected">
          <option :value="FilterMaps.Gray">灰色</option>
          <option :value="FilterMaps.Blur">模糊</option>
          <option :value="FilterMaps.Brightness">亮度</option>
          <option :value="FilterMaps.Contrast">对比度</option>
          <option :value="FilterMaps.Opacity">不透明度</option>
          <option :value="FilterMaps.Sepia">棕褐色</option>
          <option :value="FilterMaps.Saturate">饱和度</option>
          <option :value="FilterMaps.DropShadow">阴影</option>
        </select>
      </div>
      <div class="mt-2" v-if="selected !== FilterMaps.DropShadow">
        <input id="slider" type="range" v-model="sliderValue">
        <span id="current" class="ml-2">{{ currentLabel }}</span>
      </div>
      <div class="drop-shadow" v-else>
        <div>
          <label for="offsetX">offset-x</label>
          <input id="offsetX" v-model="filterList[FilterMaps.DropShadow].value[0]">
          <span class="ml-1">px</span>
        </div>
        <div>
          <label for="offsetY">offset-y</label>
          <input id="offsetY" v-model="filterList[FilterMaps.DropShadow].value[1]">
          <span class="ml-1">px</span>
        </div>
        <div>
          <label for="blurRadius">blur-radius</label>
          <input id="blurRadius" v-model="filterList[FilterMaps.DropShadow].value[2]">
          <span class="ml-1">px</span>
        </div>
        <div>
          <label for="color">color</label>
          <input id="color" type="color" v-model="filterList[FilterMaps.DropShadow].value[3]">
        </div>
      </div>
    </div>
    <div class="code">
         <pre><code>
.image:{
     filter: {{ cssLabel }}
   }</code></pre>
    </div>
  </div>
  <div v-else>
    <img class="image" src="/exm.jpg" alt="" :style="{ filter: cssLabel, width: `${props.width}px` }">
    <div class="css-label" v-if="!unShowCss">filter: {{ cssLabel }}</div>
  </div>
</template>

<style scoped>
.main {
  display: flex;
  justify-content: space-around;
}
.container {
  display: flex;
  justify-content: flex-end;
  align-items: flex-start;
  flex-direction: column;
  width: 50%;
}
img {
  width: 400px;
}
#slider {
  width: 350px;
  filter: grayscale(0);
}
pre {
  white-space: pre-wrap;
}
.code {
  background: #f8f8f8;
  border-radius: 4px;
  padding: 0 20px;
  width: 45%;
}
.drop-shadow {
  text-align: center;
  vertical-align: middle;
  display: flex;
  flex-wrap: wrap;
}
.drop-shadow > div {
  margin-left: 10px;
  margin-top: 10px;
}
.drop-shadow label {
  font-weight: bold;
}
.drop-shadow input {
  border: 1px solid #000000;
  width: 40px;
  margin-left: 10px;
}
.image {
  overflow: hidden;
}
.css-label {
  font-size: 10px;
}
</style>
