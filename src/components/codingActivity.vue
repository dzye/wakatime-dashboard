<script setup lang="ts">
import { defineProps, onMounted, ref, watch } from 'vue'
import * as echarts from 'echarts'
import dayjs from 'dayjs'
const props = defineProps({
  option: Object,
})
const getFormateDate = (date) => {
  return dayjs(date).format('YYYYMMDD')
}
const myRef = ref<any>(null)
watch(
  () => props.option,
  () => {
    drawChart()
  },
)
const getOriginData = () => {
  const chartData = props.option
  const projectContainer = {}
  chartData.forEach((element) => {
    const currentDate = getFormateDate(element.range.date)
    element.projects.forEach((project) => {
      const hours = project.total_seconds / 3600
      // 如果项目名称已经存在的话，则直接把值追加到该对象
      if (projectContainer[project.name]) {
        projectContainer[project.name][currentDate] = hours
      }
      else {
        // 不存在的话,则新加
        projectContainer[project.name] = {
          name: project.name,
          [currentDate]: project.total_seconds / 3600,
        }
      }
    })
  })
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
        show: true,
      },
      emphasis: {
        focus: 'series',
      },
      data: dates.map((date) => {
        return data[date]
      }),
    })
  })
  const option = {
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
    yAxis: {
      type: 'value',
    },
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
