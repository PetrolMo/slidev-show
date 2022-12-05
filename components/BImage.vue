<script setup lang="ts">
import { ref, Ref, computed, watch } from 'vue'

enum FilterMaps {
  Gray = 0,
  Blur = 1,
  Brightness = 2,
  Contrast = 3,
  Opacity = 4,
  Sepia = 5,
  Saturate = 6
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
  }
}
let isMultiple = ref(false)
watch(isMultiple, (val) => {
  if(!val) {
    filterList.value = JSON.parse(JSON.stringify(defaultList))
  }
})
let cssLabel:Ref<string> = ref('grayscale(0%)')
const filterList: Ref<Filter> = ref(JSON.parse(JSON.stringify(defaultList)))
const selected: Ref<FilterMaps> = ref(FilterMaps.Gray)
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
watch(selected, (val: number) => {
  const value = filterList.value[val].value ? filterList.value[val].value : filterList.value[val].default
  sliderValue.value = Number((value / filterList.value[val].max * 100).toFixed(0))
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
  selectedList.value.forEach((key: number) => {
    cssLabel.value +=  ` ${filterList.value[key].key}(${filterList.value[key].value}${filterList.value[key].unit})`
  })
})
</script>

<template>
  <div class="main">
    <div class="container">
      <img src="../public/exm.JPG" width="400" height="400" alt="" :style="{ filter: cssLabel }">
      <div class="mt-2">
        <input id="multiple" type="checkbox" v-model="isMultiple">
        <label for="multiple" class="ml-1">混合使用</label>
      </div>
      <div class="mt-2">
        <select id="filter" v-model="selected">
          <option :value="0">灰色</option>
          <option :value="1">模糊</option>
          <option :value="2">亮度</option>
          <option :value="3">对比度</option>
          <option :value="4">不透明度</option>
          <option :value="5">棕褐色</option>
          <option :value="6">饱和度</option>
        </select>
      </div>
      <div class="mt-2">
        <input id="slider" type="range" v-model="sliderValue">
        <span id="current" class="ml-2">{{ currentLabel }}</span>
      </div>
    </div>
    <div class="code">
         <pre><code>
image:{
     filter: {{ cssLabel }}
   }</code></pre>
    </div>
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
</style>
