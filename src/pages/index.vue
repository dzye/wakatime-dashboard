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
// const option = reactive({
//   title: {
//     text: '系统折线图',
//   },
//   tooltip: {
//     trigger: 'axis',
//     axisPointer: {
//       type: 'cross',
//       label: {
//         backgroundColor: '#6a7985',
//       },
//     },
//   },
//   legend: {
//     data: ['新增注册', '付费用户', '活跃用户', '订单数', '当日总收入'],
//   },
//   toolbox: {
//     feature: {
//       saveAsImage: {},
//     },
//   },
//   grid: {
//     left: '3%',
//     right: '4%',
//     bottom: '3%',
//     containLabel: true,
//   },
//   xAxis: [
//     {
//       type: 'category',
//       boundaryGap: false,
//       data: [
//         '2021-03-11',
//         '2021-03-12',
//         '2021-03-13',
//         '2021-03-14',
//         '2021-03-15',
//         '2021-03-16',
//         '2021-03-17',
//       ],
//     },
//   ],
//   yAxis: [
//     {
//       type: 'value',
//     },
//   ],
//   series: [
//     {
//       name: '新增注册',
//       type: 'line',
//       stack: '总量',
//       areaStyle: {},
//       emphasis: {
//         focus: 'series',
//       },
//       data: [120, 132, 101, 134, 90, 230, 210],
//     },
//     {
//       name: '付费用户',
//       type: 'line',
//       stack: '总量',
//       areaStyle: {},
//       emphasis: {
//         focus: 'series',
//       },
//       data: [220, 182, 191, 234, 290, 330, 310],
//     },
//     {
//       name: '活跃用户',
//       type: 'line',
//       stack: '总量',
//       areaStyle: {},
//       emphasis: {
//         focus: 'series',
//       },
//       data: [150, 232, 201, 154, 190, 330, 410],
//     },
//     {
//       name: '订单数',
//       type: 'line',
//       stack: '总量',
//       areaStyle: {},
//       emphasis: {
//         focus: 'series',
//       },
//       data: [320, 332, 301, 334, 390, 330, 320],
//     },
//     {
//       name: '当日总收入',
//       type: 'line',
//       stack: '总量',
//       label: {
//         show: true,
//         position: 'top',
//       },
//       areaStyle: {},
//       emphasis: {
//         focus: 'series',
//       },
//       data: [820, 932, 901, 934, 1290, 1330, 1320],
//     },
//   ],
// })

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
  <div class="w-100vw bg-red dark:bg-blue h-50px" />
  <div>
    <CodingActivity :option="option.option" />
  </div>
</template>
