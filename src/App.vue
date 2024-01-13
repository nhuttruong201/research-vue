<script setup>
import { onMounted, reactive, watch } from "vue";
import ResourceUsageGraph from "./components/ResourceUsageGraph.vue";

const data = reactive({
  name: "",
  currentValue: 0,
  peakValue: 0,
  recordedDateTime: "",
});

onMounted(() => {
  let index = 0;
  const interval = setInterval(() => {
    index++;
    data.name = "CPU1";
    data.currentValue = getRandomInt(0, 100);
    data.peakValue = getRandomInt(0, 100);
    data.recordedDateTime = "20:00";

    if (index >= 20) {
      clearInterval(interval);
    }
  }, 1000);
});

function getRandomInt(min, max) {
  min = Math.ceil(min);
  max = Math.floor(max);
  return Math.floor(Math.random() * (max - min) + min); // The maximum is exclusive and the minimum is inclusive
}
</script>

<template>
  <div>
    <ResourceUsageGraph
      :name="data.name"
      :current-value="data.currentValue"
      :peak-value="data.peakValue"
      :recorded-date-time="data.recordedDateTime"
    />
  </div>
</template>

<style scoped></style>
