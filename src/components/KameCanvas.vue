<script setup>
import { Chart, registerables } from "chart.js";
import { LineChart } from "vue-chart-3";
Chart.register(...registerables);

const lineData = {
  labels: ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul"],
  datasets: [
    {
      label: "My First Dataset",
      data: [65, 59, 80, 81, 56, 55, 40],
      fill: false,
      borderColor: "rgb(75, 192, 192)",
      tension: 0.1,
    },
  ],
};

</script>
<template>
  <p>Kamekusaの軌跡</p>
  <p>{{radius}}</p>
  <canvas width="600" height="300" class="canvas" id="kame-map"></canvas>
  <p><button type="button" @click="drawStart">描画スタート</button></p>
  <LineChart :chartData="lineData" />
</template>

<script>
export default {
  data() {
    return {
      x: 0,
      y: 0,
      prevX: 0,
      prevY: 0
    }
  },
  props: {
    radius: {
      type: Number,
      default: 50
    }
  },
  watch: {
    radius() {
      console.log(this.radius)
      this.draw(this.radius)
   }
  },
  methods: {
    draw(radius) {
      console.log(this.radius)
      //this.ctx.clearRect(0, 0, 600, 300)

      this.ctx.beginPath()
      this.ctx.fillStyle = "blue";
      this.ctx.lineWidth = 5;
      this.ctx.moveTo(this.prevX, this.prevY);
      this.ctx.lineTo(this.x, this.y);
      this.ctx.stroke();
      //this.ctx.fill()
      console.log(this.x)
      console.log(this.prevX)
      this.prevX = this.x
      this.prevY = this.y
    },
    drawStart() {
      let timer = setInterval(() => {
          this.x = this.x + 6
          this.y = this.y + 3
          this.draw(0)
        }
        ,1000
      ) 
      setTimeout(() => clearInterval(timer), 10000)
    }
  },
  mounted() {
    // mounted 以降で canvas の DOM にアクセスできる
    // CreateJS などを使うときにも、ここで canvas と紐付ける
    this.ctx = document.getElementById('kame-map').getContext('2d')
    this.ctx.fillStyle = "beige";
    this.ctx.fillRect(0, 0, 600, 300)
    this.draw(this.radius)
  }
}
</script>

<style scoped>
.canvas {
  border: 1px solid #000;
}
</style>
