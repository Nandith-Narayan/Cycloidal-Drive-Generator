<script setup>
import { ref, watch, onMounted } from 'vue'
const props = defineProps(['params'])
let ctx = {}
let t = 0.0
let bt = performance.now()

const numPointsToRender = ref(1000)


function render() {
  ctx.clearRect(0, 0, 500, 500)
  try {
    if (!props.params.numLobes) {
      t = 0.0
      requestAnimationFrame(render)
      return
    }
    let scaleFactor = 450 / (props.params.ringSize + props.params.pinSize / 2) // Convert mm to pixels, normalized w.r.t. drive size
    let cx = 250
    let cy = 250
    let r = props.params.ringSize / 2
    let r2 = r / (props.params.numLobes + 1)
    let r1 = r - r2
    ctx.lineWidth = 1.5

    // Draw reference circle
    ctx.beginPath()
    ctx.setLineDash([5, 10])
    ctx.arc(cx, cy, r * scaleFactor, 0, 2 * Math.PI)
    ctx.strokeStyle = 'black'
    ctx.stroke()
    ctx.closePath()

    // Draw inner circle
    ctx.beginPath()
    ctx.setLineDash([])
    ctx.arc(cx, cy, r1 * scaleFactor, 0, 2 * Math.PI)
    ctx.strokeStyle = 'black'
    ctx.stroke()
    ctx.closePath()

    // Draw outer circle
    ctx.beginPath()
    ctx.setLineDash([])
    let circle_x = r * Math.cos(t)
    let circle_y = r * Math.sin(t)
    ctx.arc(
      cx + circle_x * scaleFactor,
      cy + circle_y * scaleFactor,
      r2 * scaleFactor,
      0,
      2 * Math.PI,
    )
    ctx.strokeStyle = 'black'
    ctx.stroke()
    ctx.closePath()
    // Draw center of outer circle
    ctx.beginPath()
    ctx.setLineDash([])
    ctx.arc(
      cx + circle_x * scaleFactor,
      cy + circle_y * scaleFactor,
      2,
      0,
      2 * Math.PI,
    )
    ctx.strokeStyle = 'black'
    ctx.fillStyle = 'blue'
    ctx.stroke()
    ctx.fill()
    ctx.closePath()

    // Draw eccentric point on outer circle
    ctx.beginPath()
    ctx.setLineDash([])
    let px =
      (r1 + r2) * Math.cos(t) -
      props.params.eccentricity * Math.cos((t * (r1 + r2)) / r2)
    let py =
      (r1 + r2) * Math.sin(t) -
      props.params.eccentricity * Math.sin((t * (r1 + r2)) / r2)
    ctx.arc(cx + px * scaleFactor, cy + py * scaleFactor, 2, 0, 2 * Math.PI)
    ctx.strokeStyle = 'black'
    ctx.fillStyle = 'darkred'
    ctx.stroke()
    ctx.fill()
    ctx.closePath()

    let points = []
    // Draw reference spline
    ctx.beginPath()
    ctx.setLineDash([7, 2])
    ctx.strokeStyle = 'red'
    //ctx.moveTo();
    let thetaStep = 2*Math.PI/numPointsToRender.value
    for (let theta = 0; theta <= 2 * Math.PI; theta += thetaStep) {
      let x =
        (r1 + r2) * Math.cos(theta) -
        props.params.eccentricity * Math.cos((theta * (r1 + r2)) / r2)
      let y =
        (r1 + r2) * Math.sin(theta) -
        props.params.eccentricity * Math.sin((theta * (r1 + r2)) / r2)
      let point = {}
      point.x = x
      point.y = y
      points.push(point)
      if (theta <= t) {
        ctx.lineTo(x * scaleFactor + cx, y * scaleFactor + cy)
      }
    }
    ctx.stroke()
    ctx.closePath()

    // Draw true spline
    ctx.beginPath()
    ctx.setLineDash([])
    ctx.strokeStyle = 'darkgreen'
    //ctx.moveTo();
    for (let i = 1; i < points.length+2; i++) {
      let p1 = points[(i - 1)%points.length]
      let p2 = points[i%points.length]
      let dx = p2.x - p1.x
      let dy = p2.y - p1.y
      let len = Math.sqrt(dx * dx + dy * dy)
      let nx = -dy / len
      let ny = dx / len
      let x = p2.x + nx * (props.params.pinSize / 2)
      let y = p2.y + ny * (props.params.pinSize / 2)
      ctx.lineTo(x * scaleFactor + cx, y * scaleFactor + cy)
    }
    ctx.stroke()
    ctx.closePath()

    // Draw bearing hole, center point, and offset shaft cener point
    ctx.beginPath()
    ctx.arc(
      cx,
      cy,
      (props.params.bearingSize / 2) * scaleFactor,
      0,
      2 * Math.PI,
    )
    ctx.stroke()
    ctx.closePath()
    ctx.beginPath()
    ctx.strokeStyle = 'darkgreen'
    ctx.arc(cx, cy, 2, 0, 2 * Math.PI)
    ctx.stroke()
    ctx.closePath()
    ctx.beginPath()
    ctx.fillStyle = 'black'
    ctx.strokeStyle = 'black'
    ctx.arc(cx, cy - props.params.eccentricity * scaleFactor, 2, 0, 2 * Math.PI)
    ctx.fill()
    ctx.closePath()

    // Draw output holes on disk
    ctx.strokeStyle = 'darkgreen'
    for (let i = 0; i < props.params.numOutputPins; i++) {
      ctx.beginPath()
      let x =
        props.params.outputPinPosition *
        Math.cos((i * 2 * Math.PI) / props.params.numOutputPins)
      let y =
        props.params.outputPinPosition *
        Math.sin((i * 2 * Math.PI) / props.params.numOutputPins)
      ctx.arc(
        cx + x * scaleFactor,
        cy + y * scaleFactor,
        (props.params.eccentricity + props.params.outputPinSize / 2) *
          scaleFactor,
        0,
        2 * Math.PI,
      )
      ctx.stroke()
      ctx.closePath()
    }

    // Draw dimention label
    ctx.beginPath()
    ctx.strokeStyle = 'black'
    ctx.fillStyle = 'black'
    ctx.moveTo(cx - r * scaleFactor, 490)
    ctx.lineTo(cx + r * scaleFactor, 490)
    ctx.moveTo(cx - r * scaleFactor, 480)
    ctx.lineTo(cx - r * scaleFactor, 500)
    ctx.moveTo(cx + r * scaleFactor, 480)
    ctx.lineTo(cx + r * scaleFactor, 500)
    ctx.fillText(props.params.ringSize + ' mm', cx - 15, 500)
    ctx.stroke()
    ctx.closePath()
  } catch (e) {
    console.log(e)
  }
  if (t > 2 * Math.PI) {
    t -= 2 * Math.PI
  }
  t += (performance.now() - bt) / 1000.0

  bt = performance.now()
  requestAnimationFrame(render)
}
onMounted(() => {
  let c = document.getElementById('disk-creation')
  ctx = c.getContext('2d')
  requestAnimationFrame(render)
})
</script>

<template>
  <div id="disk-creation-div" class="card">
    <canvas id="disk-creation" width="500" height="500"></canvas><br />
    Render Resolution: {{numPointsToRender}} Points<br />
    <input v-model.number="numPointsToRender" type="range" min="100" max="10000" value="1000" step="10"/>
  </div>
</template>

<style scoped>
div#disk-creation-div {
  color: var(--fg-color-primary);
  padding: 0.5em;
}

input{
  width: 100%;
  padding: 0;
  margin: 0;
}

canvas#disk-creation {
  max-width: 100%;
  height: auto;
}
</style>
