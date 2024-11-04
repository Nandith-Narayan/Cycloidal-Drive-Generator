<script setup>
import { ref } from 'vue'
import Drawing from 'dxf-writer'
const props = defineProps(['params'])

const numPointsToExport = ref(1000)
let diskColors = [
  Drawing.ACI.BLUE,
  Drawing.ACI.RED,
  Drawing.ACI.GREEN,
  Drawing.ACI.MAGENTA,
  Drawing.ACI.CYAN,
]
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
let files = ref([])

function generateDisks() {
  files.value = []

  let numDisks = props.params.numDisks
  allDiskPoints = []
  for (let i = 0; i < numDisks; i++) {
    let phaseOffset = (i * 2 * Math.PI) / numDisks
    allDiskPoints.push(generateDiskPoints(phaseOffset))
  }

  let fullDrawing = new Drawing()
  fullDrawing.setUnits('Millimeters')

  for (let i = 0; i < numDisks; i++) {
    let phaseOffset = (i * 2 * Math.PI) / numDisks

    let color = diskColors[i % diskColors.length]
    fullDrawing.addLayer('disk_' + (i + 1).toString(), color, 'CONTINUOUS')
    fullDrawing.setActiveLayer('disk_' + (i + 1).toString())
    drawDiskOntoDXF(fullDrawing, allDiskPoints[i])
    drawHolesOntoDXF(fullDrawing, phaseOffset)

    let drawing = new Drawing()
    drawing.setUnits('Millimeters')
    drawing.addLayer('disk', Drawing.ACI.GREEN, 'CONTINUOUS')
    drawing.setActiveLayer('disk')
    drawDiskOntoDXF(drawing, allDiskPoints[i])
    drawHolesOntoDXF(drawing, phaseOffset)

    let file = new Blob([drawing.toDxfString()], { type: 'text/plain' })
    let url = URL.createObjectURL(file)
    files.value.push({
      file: file,
      url: url,
      text: 'Disk ' + (i + 1).toString(),
      filename: 'Disk ' + (i + 1).toString() + '.dxf',
    })
  }
  let ringDrawing = new Drawing()
  ringDrawing.setUnits('Millimeters')

  drawAllPins(ringDrawing)
  drawAllPins(fullDrawing)

  let pinFile = new Blob([ringDrawing.toDxfString()], { type: 'text/plain' })
  let pinUrl = URL.createObjectURL(pinFile)
  files.value.push({
    file: pinFile,
    url: pinUrl,
    text: 'Outer Pins & Output Pins Only',
    filename: 'Stationary and moving Pins.dxf',
  })

  let file = new Blob([fullDrawing.toDxfString()], { type: 'text/plain' })
  let url = URL.createObjectURL(file)
  files.value.push({
    file: file,
    url: url,
    text: 'Entire Drive',
    filename: 'Cycloidal Drive.dxf',
  })
}

function drawDiskOntoDXF(drawing, points) {
  let pointArr = []
  for (let i = 0; i < points.length; i++) {
    pointArr.push([points[i].x, points[i].y])
  }
  pointArr.push([points[0].x, points[0].y])
  drawing.drawPolyline(pointArr, true)
}

function drawHolesOntoDXF(drawing, phaseOffset) {
  for (let i = 0; i < props.params.numOutputPins; i++) {
    let x =
      props.params.eccentricity * Math.cos(phaseOffset) +
      props.params.outputPinPosition *
        Math.cos((i * 2 * Math.PI) / props.params.numOutputPins)
    let y =
      props.params.eccentricity * Math.sin(phaseOffset) +
      props.params.outputPinPosition *
        Math.sin((i * 2 * Math.PI) / props.params.numOutputPins)
    let r = props.params.eccentricity + props.params.outputPinSize / 2
    drawing.drawCircle(x, y, r)
  }

  let x = props.params.eccentricity * Math.cos(phaseOffset)
  let y = props.params.eccentricity * Math.sin(phaseOffset)
  let r = props.params.bearingSize / 2
  drawing.drawCircle(x, y, r)
}

function drawAllPins(drawing) {
  drawing.addLayer('outer_ring', Drawing.ACI.WHITE, 'CONTINUOUS')
  drawing.setActiveLayer('outer_ring')
  for (let i = 0; i < props.params.numLobes + 1; i++) {
    let x =
      (props.params.ringSize / 2) *
      Math.cos((2 * Math.PI * i) / (props.params.numLobes + 1))
    let y =
      (props.params.ringSize / 2) *
      Math.sin((2 * Math.PI * i) / (props.params.numLobes + 1))
    let r = props.params.pinSize / 2
    drawing.drawCircle(x, y, r)
  }

  drawing.addLayer('output_pins', Drawing.ACI.WHITE, 'CONTINUOUS')
  drawing.setActiveLayer('output_pins')
  for (let i = 0; i < props.params.numOutputPins; i++) {
    let x =
      props.params.outputPinPosition *
      Math.cos((i * 2 * Math.PI) / props.params.numOutputPins)
    let y =
      props.params.outputPinPosition *
      Math.sin((i * 2 * Math.PI) / props.params.numOutputPins)
    let r = props.params.outputPinSize / 2
    drawing.drawCircle(x, y, r)
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

    point.x =
      x * cos - y * sin + props.params.eccentricity * Math.cos(phaseOffset)
    point.y =
      x * sin + y * cos + props.params.eccentricity * Math.sin(phaseOffset)
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
    <h3 v-if="files.length > 0">Download Links:</h3>
    <ul>
      <li v-for="file in files">
        <a :href="file.url" :download="file.filename">{{ file.text }}</a>
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
