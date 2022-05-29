// https://echarts.apache.org/examples/zh/editor.html?c=custom-profile
<script setup lang="ts">
import { defineProps, ref, watch } from 'vue'
import * as echarts from 'echarts'
import dayjs from 'dayjs'
const props = defineProps({
  option: Object,
})
const getFormateDate = (date) => {
  return dayjs(date).format('YYYY/MM/DD')
}
const myRef = ref<any>(null)
watch(
  () => props.option,
  () => {
    drawChart()
  },
)

const dateTotal = {}
const getOriginData = () => {
  const chartData = props.option
  console.log(chartData)
  const projectContainer = {}
  chartData.forEach((element) => {
    const currentDate = getFormateDate(element.range.date)
    if (!dateTotal[currentDate])
      dateTotal[currentDate] = 0

    element.projects.forEach((project) => {
      const second = project.total_seconds
      dateTotal[currentDate] = dateTotal[currentDate] + second
      // 如果项目名称已经存在的话，则直接把值追加到该对象
      if (projectContainer[project.name]) {
        projectContainer[project.name][currentDate] = second
        projectContainer[project.name].formatTime = formatSecond(project.total_seconds)
      }
      else {
        // 不存在的话,则新加
        projectContainer[project.name] = {
          name: project.name,
          formatTime: formatSecond(project.total_seconds),
          [currentDate]: project.total_seconds,
        }
      }
    })
  })
  console.log(dateTotal)
  return Object.keys(projectContainer).map((name) => {
    return projectContainer[name]
  })
}
const getFields = () => {
  const chartData = props.option
  return chartData.map((data) => {
    return getFormateDate(data.range.date)
  })
}
/**
 * 格式化秒
 * @param result
 * @returns {string}
 */
const formatSecond = (result) => {
  const h = Math.floor((result / 3600) % 24)
  const m = Math.floor((result / 60) % 60)
  const s = Math.floor(result % 60)
  result = `${s}秒`
  if (m > 0)
    result = `${m}分钟${result}`

  if (h > 0)
    result = `${h}小时${result}`

  return result
}
// 绘制折线图
const drawChart = () => {
  // 初始化echarts实例
  const Chart = echarts.init(myRef.value)
  const originData = getOriginData()
  const dates = getFields()
  const echertsData = []
  originData.forEach((data) => {
    const projectName = data.name
    echertsData.push({
      name: projectName,
      type: 'bar',
      stack: 'total',
      label: {
        show: false,
      },
      emphasis: {
        focus: 'series',
      },
      data: dates.map((date) => {
        return data[date]
      }),
    })
  })
  console.log(dateTotal)
  echertsData.unshift({
    name: 'Total',
    type: 'line',
    yAxisIndex: 1,
    tooltip: {
      valueFormatter(value) {
        return `formatSecond(value)`
      },
    },
    data: Object.values(dateTotal),
  })
  const option = {
    tooltip: {
      trigger: 'axis',
      axisPointer: {
        type: 'shadow',
      },
      formatter: (params) => {
        let res = `${params[0].name} <br/>`
        for (const item of params) {
          if (item.value) {
            if (item.seriesName === 'Total')
              res += `<span style="font-size:18px;font-weight:500;"> ${item.seriesName} ：${formatSecond(item.value)}</span><br/>`

            else
              res += `<span style="background: ${item.color}; height:10px; width: 10px; border-radius: 50%;display: inline-block;margin-right:10px;"></span> ${item.seriesName} ：${formatSecond(item.value)}<br/>`
          }
        }
        return res
      },
    },
    legend: {},
    grid: {
      left: '3%',
      right: '4%',
      bottom: '3%',
      containLabel: true,
    },
    xAxis: {
      type: 'category',
      data: dates,
    },
    yAxis: [{
      type: 'value',
      axisLine: {
        show: false,
      },
      axisLabel: { show: false },
      axisTick: {
        show: false,
      },
    }, {
      type: 'value',
      axisLine: {
        show: false,
      },
      axisLabel: { show: false },
      axisTick: {
        show: false,
      },
    }],
    series: echertsData,
  }
  // 父组件传来的实例参数
  Chart.setOption(option)
  window.addEventListener('resize', () => {
    // 页面大小变化后Echarts也更改大小
    Chart.resize()
  })
}
</script>

<template>
  <div>
    <div ref="myRef" :style="{ width: '100%', height: '300px' }" />
  </div>
</template>
