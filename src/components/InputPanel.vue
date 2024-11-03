<script setup>
import { ref, watch } from 'vue'
const emit = defineEmits(['updateParams'])
const numLobes = ref(24) //24
const ringSize = ref(70) //70
const eccentricity = ref(0.7)
const bearingSize = ref(22.2)
const pinSize = ref(3.5)
const numOutputPins = ref(5)
const outputPinSize = ref(5)
const outputPinPosition = ref(25)
const numDisks = ref(2)

const params = {}

params.numLobes = numLobes.value
params.ringSize = ringSize.value
params.eccentricity = eccentricity.value
params.bearingSize = bearingSize.value
params.pinSize = pinSize.value
params.numOutputPins = numOutputPins.value
params.outputPinSize = outputPinSize.value
params.outputPinPosition = outputPinPosition.value
params.numDisks = numDisks.value

params.hasChanged = true

watch(
  [
    numLobes,
    ringSize,
    eccentricity,
    bearingSize,
    pinSize,
    numOutputPins,
    outputPinSize,
    outputPinPosition,
    numDisks,
  ],
  (newVals, oldVals) => {
    params.numLobes = Math.round(numLobes.value)
    params.ringSize = ringSize.value
    params.eccentricity = eccentricity.value
    params.bearingSize = bearingSize.value
    params.pinSize = pinSize.value
    params.numOutputPins = numOutputPins.value
    params.outputPinSize = outputPinSize.value
    params.outputPinPosition = outputPinPosition.value
    params.numDisks = Math.round(Math.max(numDisks.value, 1))

    params.hasChanged = true
    emit('updateParams', params)
  },
)

emit('updateParams', params)
</script>
<template>
  <div class="card">
    <ul>
      <li>
        Number of Lobes: {{ params.numLobes }}
        <div class="tooltip">
          <img src="../assets/question_mark.svg" />
          <span class="tooltiptext">
            If the number of lobes of the disk is N, then the effective gear
            ratio is N:1. That is, for very N rotations of the input, the output
            will rotate once.
          </span>
        </div>
        <br />
        <input v-model.number="numLobes" />
      </li>
      <li>
        Drive size: {{ params.ringSize }} mm
        <div class="tooltip">
          <img src="../assets/question_mark.svg" />
          <span class="tooltiptext">
            This is the size of the cyclodial drive. The outer pins are placed
            in a circle with the diameter equal to the drive size.
          </span>
        </div>
        <input v-model.number="ringSize" />
      </li>
      <li>
        Eccentricity: {{ params.eccentricity }} mm
        <div class="tooltip">
          <img src="../assets/question_mark.svg" />
          <span class="tooltiptext">
            This is the offset between the center of the drive and the center of
            the cycloidal disk. Drives with higher eccentricity can handle more
            torque before slipping, but require tighter tolerences.
          </span>
        </div>
        <input v-model.number="eccentricity" />
      </li>
      <li>
        Bearing Diameter: {{ params.bearingSize }} mm
        <div class="tooltip">
          <img src="../assets/question_mark.svg" />
          <span class="tooltiptext">
            This is the diameter of the center hole in the cycloidal disk. A
            normal bearing can be placed in this hole, and an off-center shaft
            can be inserted into the bearing to provide the required
            eccentricity.
          </span>
        </div>
        <input v-model.number="bearingSize" />
      </li>
      <li>
        Pin Diameter: {{ params.pinSize }} mm
        <div class="tooltip">
          <img src="../assets/question_mark.svg" />
          <span class="tooltiptext"> Diameter of the outer pins. </span>
        </div>
        <input v-model.number="pinSize" />
      </li>
      <li>
        Number of Output Pins: {{ params.numOutputPins }}
        <div class="tooltip">
          <img src="../assets/question_mark.svg" />
          <span class="tooltiptext"> Number of pins of the output shaft. </span>
        </div>
        <input v-model.number="numOutputPins" />
      </li>
      <li>
        Output Pin Diameter: {{ params.outputPinSize }} mm
        <div class="tooltip">
          <img src="../assets/question_mark.svg" />
          <span class="tooltiptext"> Diameter of the output shaft pins. </span>
        </div>
        <input v-model.number="outputPinSize" />
      </li>
      <li>
        Output Pin Distance from Center: {{ params.outputPinPosition }} mm
        <div class="tooltip">
          <img src="../assets/question_mark.svg" />
          <span class="tooltiptext">
            This is the distance of the center of the output shaft pins to the
            center of the drive.
          </span>
        </div>
        <input v-model.number="outputPinPosition" />
      </li>
      <li>
        Number of Disks: {{ params.numDisks }}
        <div class="tooltip">
          <img src="../assets/question_mark.svg" />
          <span class="tooltiptext">
            This is the number of disks to stack. By adding disks that are
            offset, the drive can be balanced. More disks can improve the amount
            of torque the drive can handle before failing.
          </span>
        </div>
        <input v-model.number="numDisks" />
      </li>
    </ul>
  </div>
</template>

<style scoped>
ul {
  list-style: none;
  padding-left: 0;
  color: var(--fg-color-primary);
}
li {
}
input {
  padding: 0;
  margin: 0;
  width: 99%;
  border-width: 1px;
  border-style: solid;
  color: var(--fg-color-secondary);
  background: var(--bg-color-secondary);
}
img {
  width: 1em;
  margin-left: 0.2em;
  padding: 0;
  margin-bottom: 0;
  margin-top: 0;
}

.tooltip {
  position: relative;
  display: inline-block;
}

.tooltip .tooltiptext {
  visibility: hidden;
  width: 10em;
  background-color: black;
  color: #fff;
  text-align: center;
  padding: 1em;
  border-radius: 6px;

  /* Position the tooltip text - see examples below! */
  position: absolute;
  z-index: 1;
}
.tooltip:hover .tooltiptext {
  visibility: visible;
}
</style>
