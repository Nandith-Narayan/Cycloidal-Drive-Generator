<script setup>
import { ref, watch, onMounted } from 'vue'
const props = defineProps(['params'])
let ctx = {}
let t = 0.0
let bt = performance.now()

let diskColors = ['darkblue', 'darkred', 'darkgreen']
let diskPoints = []
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

    drawPins(cx, cy, r, scaleFactor)

    //console.log(t/(props.params.numLobes), diskPhase);
    if (props.params.hasChanged) {
    diskPoints = []
    }


    let diskPhase = -t / props.params.numLobes
    drawOutputPins(cx, cy, scaleFactor, diskPhase)
    let numDisks = 2
    for (let i = 0; i < numDisks; i++) {
      let color = diskColors[i % diskColors.length]
      let phaseOffset = (i * 2 * Math.PI) / numDisks
      if (props.params.hasChanged) {
        diskPoints.push(generateDiskPoints(phaseOffset))
      }

      drawDisk(
        diskPoints[i],
        cx +
          props.params.eccentricity * Math.cos(t + phaseOffset) * scaleFactor,
        cy +
          props.params.eccentricity * Math.sin(t + phaseOffset) * scaleFactor,
        scaleFactor,
        diskPhase,
        phaseOffset,
        color,
      )
    }
    if (props.params.hasChanged) {
    props.params.hasChanged = false
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
  if (t > 2 * Math.PI * props.params.numLobes) {
    t -= 2 * Math.PI * props.params.numLobes
  }
  t += (5 * (performance.now() - bt)) / 1000.0
  bt = performance.now()
  requestAnimationFrame(render)
}
function generateDiskPoints(phaseOffset) {
  let points = []
  let diskPoints = []

  let r = props.params.ringSize / 2
  let r2 = r / (props.params.numLobes + 1)
  let r1 = r - r2

  let thetaStepSize = 0.001

  for (let theta = 0; theta <= 2 * Math.PI; theta += thetaStepSize) {
    let angle = theta
    let x =
      (r1 + r2) * Math.cos(angle) -
      props.params.eccentricity * Math.cos((angle * (r1 + r2)) / r2)
    let y =
      (r1 + r2) * Math.sin(angle) -
      props.params.eccentricity * Math.sin((angle * (r1 + r2)) / r2)
    let point = {}
    let rotateAngle = -phaseOffset/props.params.numLobes
    let cos = Math.cos(rotateAngle)
    let sin = Math.sin(rotateAngle)
    point.x = x * cos - y * sin
    point.y = x * sin + y * cos
    points.push(point)
  }
  let minOffset = Number.MAX_VALUE
  let maxOffset = -1
  for (let i = 1; i < points.length; i++) {
    let p1 = points[i - 1]
    let p2 = points[i]
    let dx = p2.x - p1.x
    let dy = p2.y - p1.y
    let len = Math.sqrt(dx * dx + dy * dy)
    let nx = -dy / len
    let ny = dx / len
    let x = p2.x + nx * (props.params.pinSize / 2)
    let y = p2.y + ny * (props.params.pinSize / 2)
    let point = {}
    point.x = x
    point.y = y
    diskPoints.push(point)
  }

  return diskPoints
}

function drawDisk(points, cx, cy, scaleFactor, phase, phaseOffset, color) {
  ctx.beginPath()
  ctx.setLineDash([])
  ctx.strokeStyle = color
  //ctx.moveTo();
  let cos = Math.cos(phase)
  let sin = Math.sin(phase)
  for (let i = 0; i < points.length; i++) {
    let px = points[i].x
    let py = points[i].y

    let x = px * cos - py * sin
    let y = px * sin + py * cos

    ctx.lineTo(x * scaleFactor + cx, y * scaleFactor + cy)
  }
  ctx.stroke()
  ctx.closePath()
  ctx.beginPath()
  ctx.fillStyle = color
  ctx.arc(cx, cy, 2, 0, 2 * Math.PI)
  ctx.fill()
  ctx.stroke()
  ctx.closePath()
  let px = points[0].x
  let py = points[0].y

  let x = px * cos - py * sin
  let y = px * sin + py * cos
  ctx.stroke()
  ctx.closePath()
  ctx.beginPath()
  ctx.fillStyle = color
  ctx.strokeStyle = 'black'
  ctx.arc(x * scaleFactor + cx, y * scaleFactor + cy, 2, 0, 2 * Math.PI)
  ctx.stroke()
  ctx.fill()
  ctx.closePath()

  ctx.strokeStyle = color
  for (let i = 0; i < props.params.numOutputPins; i++) {
    ctx.beginPath()
    let x =
      props.params.outputPinPosition *
      Math.cos(phase + (i * 2 * Math.PI) / props.params.numOutputPins)
    let y =
      props.params.outputPinPosition *
      Math.sin(phase + (i * 2 * Math.PI) / props.params.numOutputPins)
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
  }
}

function drawPins(cx, cy, r, scaleFactor) {
  for (let i = 0; i < props.params.numLobes + 1; i++) {
    ctx.beginPath()
    ctx.setLineDash([])
    ctx.arc(
      cx +
        r *
          Math.cos((2 * Math.PI * i) / (props.params.numLobes + 1)) *
          scaleFactor,
      cy +
        r *
          Math.sin((2 * Math.PI * i) / (props.params.numLobes + 1)) *
          scaleFactor,
      (props.params.pinSize / 2) * scaleFactor,
      0,
      2 * Math.PI,
    )
    ctx.strokeStyle = 'black'
    ctx.fillStyle = 'black'
    ctx.fill()
    ctx.closePath()
  }
}

function drawOutputPins(cx, cy, scaleFactor, outputPhase) {
  for (let i = 0; i < props.params.numOutputPins; i++) {
    ctx.beginPath()
    ctx.setLineDash([])
    let x =
      props.params.outputPinPosition *
      Math.cos(outputPhase + (i * 2 * Math.PI) / props.params.numOutputPins)
    let y =
      props.params.outputPinPosition *
      Math.sin(outputPhase + (i * 2 * Math.PI) / props.params.numOutputPins)

    ctx.arc(
      cx + x * scaleFactor,
      cy + y * scaleFactor,
      (props.params.outputPinSize / 2) * scaleFactor,
      0,
      2 * Math.PI,
    )
    ctx.strokeStyle = 'black'
    ctx.fillStyle = 'black'
    ctx.fill()
    ctx.closePath()
  }
}

onMounted(() => {
  let c = document.getElementById('disk-animation')
  ctx = c.getContext('2d')
  console.log('AMONGUS')
  requestAnimationFrame(render)
})
</script>

<template>
  <div id="disk-animation-div" class="card">
    <canvas id="disk-animation" width="500" height="500"></canvas>
  </div>
</template>

<style scoped>
div#disk-animation-div {
}

canvas#disk-animation {
  width: 500px;
  height: 500px;
}
</style>
