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
    params.numDisks = Math.round(numDisks.value)

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
        Number of Lobes: {{ params.numLobes }}<br />
        <input v-model.number="numLobes" />
      </li>
      <li>
        Drive size: {{ params.ringSize }} mm<br />
        <input v-model.number="ringSize" />
      </li>
      <li>
        Eccentricity: {{ params.eccentricity }} mm<br />
        <input v-model.number="eccentricity" />
      </li>
      <li>
        Bearing Diameter: {{ params.bearingSize }} mm<br />
        <input v-model.number="bearingSize" />
      </li>
      <li>
        Pin Diameter: {{ params.pinSize }} mm<br />
        <input v-model.number="pinSize" />
      </li>
      <li>
        Number of Output Pins: {{ params.numOutputPins }}<br />
        <input v-model.number="numOutputPins" />
      </li>
      <li>
        Output Pin Diameter: {{ params.outputPinSize }} mm<br />
        <input v-model.number="outputPinSize" />
      </li>
      <li>
        Output Pin Distance from Center: {{ params.outputPinPosition }} mm<br />
        <input v-model.number="outputPinPosition" />
      </li>
      <li>
        Number of Disks: {{ params.numDisks }}<br />
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
  width: 15em;
  border-width: 1px;
  border-style: solid;
  color: var(--fg-color-secondary);
  background: var(--bg-color-secondary);
}
</style>
