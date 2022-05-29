<script setup lang="ts">
import { onMounted, reactive } from 'vue'
import axios from 'axios'
import codingActivity from '@/components/codingActivity.vue'
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
  axios
    .get('https://api.github.com/gists/6530ddc327cf54a5ef01204a4cca17a5')
    .then((response: any): any => fetchSingleFile(response))
    .then((values) => {
      const data = values.reduce((sum: any, current: any) => {
        sum.push(current.data)
        return sum
      }, [])
      option.option = getLastData(data)
      const total = getTotal(option.option)
    })
})
</script>

<template>
  <div class="w-100vw bg-red dark:bg-blue h-50px">
    <input type="text">
    <button>最近7天</button>
    <button>最近30天</button>
    <button>最近90天</button>
  </div>
  <div>
    <CodingActivity :option="option.option" />
  </div>
</template>
