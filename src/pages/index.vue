<script setup lang="ts">
import { onMounted, reactive } from 'vue'
import axios from 'axios'
import codingActivity from '@/components/codingActivity.vue'
const gistId = ref(null)
const getData = () => {
  initData()
}
function initData() {
  axios
    .get(`https://api.github.com/gists/${gistId.value}`)
    .then((response: any): any => fetchSingleFile(response))
    .then((values) => {
      const data = values.reduce((sum: any, current: any) => {
        sum.push(current.data)
        return sum
      }, [])
      option.option = getLastData(data)
      const total = getTotal(option.option)
    })
}
function fetchSingleFile(response: any = {}) {
  const selectedValue = 7
  const {
    data: { files },
  } = response
  const fetchTasks: any[] = []
  const length = Object.keys(files).length - 1
  const startIndex = selectedValue >= length ? 0 : length - selectedValue
  // 选取已选中的天数
  const filesNames = Object.keys(files).slice(startIndex)
  filesNames.forEach((fileName) => {
    const { type, filename, raw_url } = files[fileName] || {}
    if (type === 'application/json' && /summaries/.test(filename))
      fetchTasks.push(axios.get(raw_url))
  })
  return Promise.all(fetchTasks)
}

function getTotal(data: any) {
  return data.reduce((x: any, y: any) => x + y.grand_total.total_seconds, 0)
}

function getLastData(data: any) {
  const length = data.length
  // const projectNames = {}
  const lineChartData = []

  for (let index = length - 1; index >= 0; index--) {
    // 每天的统计数据
    const summary = data[index].data ? data[index].data[0] : data[index][0]
    lineChartData.push(summary)
  }
  return lineChartData.reverse()
}
const option = reactive({ option: [] })

onMounted(() => {
  if (gistId.value)
    initData()
})
</script>

<template>
  <n-layout>
    <n-layout-header class="w100vw flex p2 dark:bg-#001529">
      <n-input v-model:value="gistId" class="important-w25%" />
      <n-button type="primary" class="important-w6% ml-1" @click="getData">
        <template #icon>
          <n-icon i-carbon:search />
        </template>
      </n-button>
    </n-layout-header>
    <n-layout-content content-style="padding: 24px;">
      <n-grid :x-gap="12" :y-gap="8" :cols="2">
        <n-grid-item>
          <CodingActivity :option="option.option" />
        </n-grid-item>
        <n-grid-item>
          <CodingActivity :option="option.option" />
        </n-grid-item>
        <n-grid-item>
          <CodingActivity :option="option.option" />
        </n-grid-item>
        <n-grid-item>
          <CodingActivity :option="option.option" />
        </n-grid-item>
      </n-grid>
    </n-layout-content>
    <n-layout-footer>
      <Footer />
    </n-layout-footer>
  </n-layout>
</template>
