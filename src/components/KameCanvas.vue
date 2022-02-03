<script setup>
import { Chart, registerables } from "chart.js"
import { LineChart } from "vue-chart-3"
import axios from "axios"
import { format, parseISO, toDate } from 'date-fns'
Chart.register(...registerables)
import flatPickr from 'vue-flatpickr-component';
import 'flatpickr/dist/flatpickr.css';

axios.defaults.headers.post['Content-Type'] = 'application/json;charset=utf-8'
axios.defaults.headers.post['Access-Control-Allow-Origin'] = '*'
const apiUri = 'http://localhost:7071/api/HttpExample'
const kameImg = new Image()
const houseImg = new Image()
const waterImg = new Image()
const foodImg = new Image()
const lightImg = new Image()
const kameLImg = new Image()
const kameRImg = new Image()
const kameUImg = new Image()
const kameDImg = new Image()
kameImg.src = "/src/assets/img/pan_kame.png"
houseImg.src = "/src/assets/img/house.png"
waterImg.src = "/src/assets/img/water.png"
foodImg.src = "/src/assets/img/food.png"
lightImg.src = "/src/assets/img/light.png"
kameLImg.src = "/src/assets/img/kame_left.png"
kameRImg.src = "/src/assets/img/pan_kame.png"
kameUImg.src = "/src/assets/img/kame_up.png"
kameDImg.src = "/src/assets/img/kame_down.png"

const lineData = {
  labels: [...Array(24).keys()],
  datasets: [
    {
      label: "時間あたりの移動量",
      data: Array.from({length: 24}, () => 0),
      fill: true,
      borderColor: "rgb(255, 192, 192)",
      tension: 1,
    },
  ],
}

const flatPickerConfig = {
  enableTime: false,
  dateFormat: "Y/m/d",
}

defineProps({
  msg: String
})
</script>
<template>
  <h1>{{ msg }}</h1>
  <el-row justify="center">
    <el-col :span="24">
      <flat-pickr v-model="date" :config="flatPickerConfig"></flat-pickr>の様子
    </el-col>
    <el-col :span="24">
      &nbsp;
    </el-col>
  </el-row>
  <el-row justify="center">
    <el-col :span="15">
      <canvas width="600" height="300" class="canvas" id="kame-map"></canvas>
      <h2>kamexaの軌跡</h2>
      <el-row justify="center">
        <el-col :span="24">
          <el-button type="success" @click="drawStart" size="large" v-bind:disabled="!isLoading">描画スタート</el-button>
        </el-col>
      </el-row>
    </el-col>
    <el-col :span="9">
      <LineChart :chartData="chartData" />
      <h2>Kamexaの活動量</h2>
    </el-col>
  </el-row>
</template>

<script>
export default {
  data() {
    return {
      x: 520,
      y: 50,
      direction: 'L',
      prevX: 520,
      prevY: 50,
      activityData: [],
      month: 2,
      day: 2,
      date: '2022-02-02',
      parsedData: [],
      lineData: {},
      isLoading: false
    }
  },
  computed: {
    chartData: function() {
      return this.lineData
    }
  },
  watch: {
    activityData: function() {
      this.isLoading = true
    },
    date: async function() {
      [ this.month, this.day ] = this.date.split('/').slice(1).map(v => parseInt(v))
      this.isLoading = false
      await axios.get(this.apiUri)
        .then(response => response.data.body)
        .then(body => (this.activityData = body))
      this.drawPrepare()
    },
  },
  methods: {
    draw() {
      this.ctx.strokeStyle = "rgb(192, 192, 192)"
      this.ctx.lineWidth = 1
      this.ctx.moveTo(this.prevX, this.prevY)
      this.ctx.lineTo(this.x, this.y)
      this.ctx.stroke()
      
      if (this.direction === 'L'){ this.ctx.drawImage(this.kameLImg, this.x, this.y, 64, 64) }
      else if (this.direction === 'R'){ this.ctx.drawImage(this.kameRImg, this.x, this.y, 64, 64) }
      else if (this.direction === 'U'){ this.ctx.drawImage(this.kameUImg, this.x, this.y, 64, 64) }
      else if (this.direction === 'D'){ this.ctx.drawImage(this.kameDImg, this.x, this.y, 64, 64) }
      else{ this.ctx.drawImage(this.kameLImg, this.x, this.y, 64, 64) }

      this.prevX = this.x
      this.prevY = this.y
    },
    drawPrepare() {
      this.ctx = document.getElementById('kame-map').getContext('2d')
      this.ctx.beginPath()
      this.ctx.fillStyle = "beige"
      this.ctx.fillRect(0, 0, 600, 300)
      this.ctx.font = "bold 18px sans-serif"

      let _activityData = JSON.parse(JSON.stringify(this.activityData))

      const xmax = _activityData.reduce((p, c) => p.X > c.X ? p : c)
      const xmin = _activityData.reduce((p, c) => p.X < c.X ? p : c)
      const ymax = _activityData.reduce((p, c) => p.Y > c.Y ? p : c)
      const ymin = _activityData.reduce((p, c) => p.Y < c.Y ? p : c)
      console.log(format(new Date(), "y/M/d"))
      console.log(this.month)
      console.log(this.day)

      let _moved = Array.from({length: 24}, () => 0)
      this.parsedData = []
      _activityData.forEach((v) => {
        let day = format(toDate(parseISO(v.Timestamp)), "d")
        let month = format(toDate(parseISO(v.Timestamp)), "M")
        if( day != this.day || month != this.month ){ return; }
        let hour = format(toDate(parseISO(v.Timestamp)), "H")
        let min = format(toDate(parseISO(v.Timestamp)), "m")

        if (typeof this.parsedData[hour] === 'undefined'){
          this.parsedData[hour] = []
        }
        if (typeof this.parsedData[hour][min] === 'undefined'){
          this.parsedData[hour][min] = [{x: v.X, y: v.Y, direction: v.Direction}]
        }else{
          this.parsedData[hour][min].push({x: v.X, y: v.Y, direction: v.Direction})
        }
        _moved[hour] += parseInt(v.Moved*10)
      })
      this.lineData.datasets[0].data = _moved

      this.ctx.drawImage(this.houseImg, 470 , 30, 150, 150)
      this.ctx.drawImage(this.foodImg, 0 , 20, 120, 120)
      this.ctx.drawImage(this.waterImg, 0 , 170, 120, 120)
      this.ctx.drawImage(this.lightImg, 400 , 230, 80, 80)
    },
    async drawStart() {
      // 1日あたり180*24=4440ポイントの再生
      const timerapps = []
      let i = 0
      let hour = 1
      let min = 1
      let timer = setInterval(() => {
          if ( i === 0 ){
            this.prevX = 520
            this.prevY = 50
            this.x = 520
            this.y = 50
            this.direction = 'L' 
            this.draw()
          }
          if (typeof this.parsedData[hour.toString()] !== 'undefined' &&
              typeof this.parsedData[hour.toString()][min.toString()] !== 'undefined' &&
              typeof this.parsedData[hour.toString()][min.toString()][i%3] !== 'undefined' )
          {
            this.x = Math.round((this.parsedData[hour.toString()][min.toString()][i%3].x - 0.055) * 4000, 2)
            this.y = 250 - Math.round((this.parsedData[hour.toString()][min.toString()][i%3].y + 0.001) * 1500, 2)
            if (this.x < 25) this.x = 25
            if (this.x > 575) this.x = 575
            if (this.y < 25) this.y = 25
            if (this.y > 275) this.y = 275
            this.direction = this.parsedData[hour.toString()][min.toString()][i%3].direction ?? 'L'
            this.draw()
          }
          if (i % 3 === 0){
            min++
          }
          if (min === 60 ) {
            min = 0
            hour++
          }
          if (hour <= 23 ) {
           this.ctx.fillRect(0, 274, 65, 300)
           this.ctx.fillStyle = "darkgreen"
           this.ctx.fillText(`${hour}:${min}`, 5, 294)
           this.ctx.fillStyle = "beige"
          }
          i++
        }
        ,10
      ) 
      setTimeout(() => clearInterval(timer), 10*4440)
    }
  },
  async mounted() {
    await axios.get(this.apiUri)
      .then(response => response.data.body)
      .then(body => (this.activityData = body))

    this.drawPrepare()
    this.draw()
  }
}
</script>

<style scoped>
.canvas {
  border: 1px solid #000;
}
</style>
