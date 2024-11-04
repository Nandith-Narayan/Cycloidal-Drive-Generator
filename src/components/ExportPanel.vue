<script setup>
import { ref } from 'vue'
import Drawing from 'dxf-writer'
const props = defineProps(['params'])

const numPointsToExport = ref(1000)

let allDiskPoints = []

/*let d = new Drawing()

d.setUnits('Millimeters')
d.drawText(10, 0, 10, 0, 'Hello World') // draw text in the default layer named "0"
d.addLayer('l_green', Drawing.ACI.GREEN, 'CONTINUOUS')
d.setActiveLayer('l_green')
d.drawText(20, -70, 10, 0, 'go green!')

//or fluent
d.addLayer('l_yellow', Drawing.ACI.YELLOW, 'DOTTED')
  .setActiveLayer('l_yellow')
  .drawCircle(50, -30, 25)

console.log(d.toDxfString())*/
let files = []
let urls = ref([])

function generateDisks() {
  files = []
  urls.value = []

  let numDisks = props.params.numDisks
  allDiskPoints = [];
  for (let i = 0; i < numDisks; i++) {
    let phaseOffset = (i * 2 * Math.PI) / numDisks
    allDiskPoints.push(generateDiskPoints(phaseOffset))
  }

  for (let i = 0; i < numDisks; i++) {
    let drawing = new Drawing()
    drawing.setUnits('Millimeters')
    drawing.addLayer('disk', Drawing.ACI.GREEN, 'CONTINUOUS')
    drawing.setActiveLayer('disk')
    let pointArr = []
    for(let j=0;j<allDiskPoints[i].length;j++){
      pointArr.push([allDiskPoints[i][j].x, allDiskPoints[i][j].y])
    }
    pointArr.push([allDiskPoints[i][0].x, allDiskPoints[i][0].y])
    drawing.drawPolyline(pointArr, true, 1, 1)

    let file = new Blob([drawing.toDxfString()], {type: 'text/plain'})
    let url = URL.createObjectURL(file)
    files.push(file)
    urls.value.push(url)
  }



}

function generateDiskPoints(phaseOffset) {
  let points = []
  let diskPoints = []

  let r = props.params.ringSize / 2
  let r2 = r / (props.params.numLobes + 1)
  let r1 = r - r2

  let thetaStepSize = (2 * Math.PI) / numPointsToExport.value

  for (let theta = 0; theta <= 2 * Math.PI; theta += thetaStepSize) {
    let angle = theta + phaseOffset / props.params.numLobes
    let x =
      (r1 + r2) * Math.cos(angle) -
      props.params.eccentricity * Math.cos((angle * (r1 + r2)) / r2)
    let y =
      (r1 + r2) * Math.sin(angle) -
      props.params.eccentricity * Math.sin((angle * (r1 + r2)) / r2)
    let point = {}
    let rotateAngle = -phaseOffset / props.params.numLobes
    let cos = Math.cos(rotateAngle)
    let sin = Math.sin(rotateAngle)
    point.x = x * cos - y * sin
    point.y = x * sin + y * cos
    points.push(point)
  }

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
</script>

<template>
  <div id="disk-export-div" class="card">
    Export Resolution: {{ numPointsToExport }} Points<br />
    <input
      v-model.number="numPointsToExport"
      type="range"
      min="500"
      max="10000"
      value="1000"
      step="100"
    />

    <button @click="generateDisks">
      Export Disk{{ props.params.numDisks > 1 ? 's' : '' }}
    </button>
    <ul>
    <li v-for="url in urls">
  {{ url }}
</li>
    </ul>
  </div>
</template>

<style>
div#disk-export-div {
  color: var(--fg-color-primary);
  padding: 0.5em;
}

input {
  width: 100%;
  padding: 0;
  margin: 0;
}
</style>
