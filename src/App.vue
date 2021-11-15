<template>
  <div style="">
    <input type="file" ref="file" @change="loadFile" id="file">
    <div style="width: 600px;height: 400px;" ref="chart"></div>
    <div ref="table">
      <table border="1" cellpadding="0" cellspacing="0" style="border-collapse:collapse;">
        <tbody>
        <tr>
          <td>Frequency(GHz)</td>
          <td v-for="item in tableData.slice(0, 5)" :key="item[0]">
            {{item[0]}}
          </td>
        </tr>
        <tr>
          <td>{{dataName}}</td>
          <td v-for="item in tableData.slice(0, 5)" :key="item[1]">
            {{item[1]}}
          </td>
        </tr>

        <tr>
          <td>Frequency(GHz)</td>
          <td v-for="item in tableData.slice(5, 10)" :key="item[0]">
            {{item[0]}}
          </td>
        </tr>

        <tr>
          <td>{{dataName}}</td>
          <td v-for="item in tableData.slice(5, 10)" :key="item[1]">
            {{item[1]}}
          </td>
        </tr>

        <tr>
          <td>Frequency(GHz)</td>
          <td v-for="item in tableData.slice(10, 15)" :key="item[0]">
            {{item[0]}}
          </td>
        </tr>

        <tr>
          <td>{{dataName}}</td>
          <td v-for="item in tableData.slice(10, 15)" :key="item[1]">
            {{item[1]}}
          </td>
        </tr>

        <tr>
          <td>Frequency(GHz)</td>
          <td v-for="item in tableData.slice(15, 20)" :key="item[0]">
            {{item[0]}}
          </td>
        </tr>

        <tr>
          <td>{{dataName}}</td>
          <td v-for="item in tableData.slice(15, 20)" :key="item[1]">
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
      dataName: 'N/A'
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
    generateTable (data, numPoints, dataName) {
      // choose 20 data, slice to 5 lines
      const num = Math.ceil(numPoints / 20)
      this.tableData = data.reduce((acc, cur, idx) => {
        if (idx % num === 0) {
          acc.push([cur[0].toFixed(2), cur[1].toFixed(2)])
        }
        return acc
      }, [])
      this.dataName = dataName
    },
    parse (s) {
      // const lines = this.$refs.data.value.split('\n')
      const lines = s.split('\r\n')
      // get title
      const title = /参数\s(.*)$/.exec(lines[3])[1]
      const numPoints = parseInt(/扫描点数\s(\d+)$/.exec(lines[7])[1])
      const dataName = /数据格式\s(.*)$/.exec(lines[8])[1]
      const data = lines.slice(11, numPoints + 11).map(v => {
        const arr = v.split('    ')
        return [eval(arr[0]) / 1e9, eval(arr[1])]
      })
      this.generateTable(data, numPoints, dataName)
      const chart = echarts.init(this.$refs.chart)
      chart.setOption({
        title: {
          text: title
        },
        tooltip: {},
        legend: {
          data: [dataName]
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
          name: dataName,
          type: 'line',
          showSymbol: false,
          data
        }]
      })
    }
  }
}
</script>

<style scoped>
</style>
