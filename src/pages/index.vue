<script setup lang="ts">
import { onMounted } from 'vue'
import axios from 'axios'
function fetchSingleFile(response: any = {}) {
  const selectedValue = 99
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

// function getTotal(data: any) {
//   return data.reduce((x: any, y: any) => x + y.grand_total.total_seconds, 0)
// }

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
onMounted(() => {
  axios
    .get('https://api.github.com/gists/75f3b2ec23c7f69594ca3d9e8b7ea81d')
    .then((response: any): any => fetchSingleFile(response))
    .then((values) => {
      const data = values.reduce((sum: any, current: any) => {
        sum.push(current.data)
        return sum
      }, [])
      const chartData = getLastData(data)
      console.log(chartData)
    })
})
</script>

<template>
  <div />
</template>
