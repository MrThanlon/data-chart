<template>
  <div style="">
    <input type="file" ref="file" @change="loadFile" id="file">
    <div style="width: 600px;height: 400px;" ref="chart"></div>
    <div ref="table">
      <p>表格生成</p>
      <label>
        起始
        <input type="number" v-model="start">
      </label>
      <label>
        步进
        <input type="number" v-model="step">
      </label>
      <label>
        结束
        <input type="number" v-model="end">
      </label>
      <button @click="generateTable">生成</button>
      <table border="1" cellpadding="0" cellspacing="0" style="border-collapse:collapse;margin-top: 20px;">
        <tbody>
        <tr>
          <td>频率(GHz)</td>
          <td v-for="item in tableData" :key="item[0]">
            {{item[0]}}
          </td>
        </tr>
        <tr>
          <td>{{title}} {{dataName}}</td>
          <td v-for="item in tableData" :key="item[1]">
            {{item[1]}}
          </td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import iconv from 'iconv-lite'

export default {
  name: 'App',
  data () {
    return {
      tableData: [],
      dataName: 'N/A',
      start: 0.1,
      step: 0.05,
      end: 3,
      data: null,
      title: ''
    }
  },
  methods: {
    async loadFile () {
      const fileElement = this.$refs.file
      if (fileElement.files.length <= 0) {
        return
      }
      const fileBlob = fileElement.files[0]
      const fileArrayBuffer = await fileBlob.arrayBuffer()
      const fileString = iconv.decode(Buffer.from(fileArrayBuffer), 'gb2312')
      // parse
      this.parse(fileString)
    },
    generateTable () {
      const start = parseFloat(this.start)
      const step = parseFloat(this.step)
      const end = parseFloat(this.end)
      let k = 0
      this.tableData = this.data.reduce((acc, cur, idx) => {
        if (cur[0] >= start + step * k && cur[0] < end + step) {
          k += 1
          acc.push([cur[0].toFixed(2), cur[1].toFixed(2)])
        }
        return acc
      }, [])
    },
    parse (s) {
      // const lines = this.$refs.data.value.split('\n')
      const lines = s.split('\r\n')
      // get title
      this.title = /参数\s(.*)$/.exec(lines[3])[1]
      const numPoints = parseInt(/扫描点数\s(\d+)$/.exec(lines[7])[1])
      this.dataName = /数据格式\s(.*)$/.exec(lines[8])[1]
      this.data = lines.slice(11, numPoints + 11).map(v => {
        const arr = v.split('    ')
        return [parseFloat(arr[0]) / 1e9, parseFloat(arr[1])]
      })
      this.generateTable()
      const chart = echarts.init(this.$refs.chart)
      chart.setOption({
        title: {
          text: this.title
        },
        tooltip: {},
        legend: {
          data: [this.dataName]
        },
        xAxis: {
          type: 'value',
          min: 'dataMin',
          max: 'dataMax',
          name: 'Frequency(GHz)',
          nameLocation: 'middle',
          axisPointer: {
            show: true
          }
        },
        yAxis: {
        },
        dataZoom: [
          {
            type: 'slider',
            start: 0,
            end: 100
          }
        ],
        toolbox: {
          feature: {
            saveAsImage: {}
          }
        },
        series: [{
          name: this.dataName,
          type: 'line',
          showSymbol: false,
          data: this.data
        }]
      })
    }
  }
}
</script>

<style scoped>
</style>
