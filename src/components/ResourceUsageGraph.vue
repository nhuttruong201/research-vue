<script setup>
import { onMounted, reactive } from "vue";

const state = reactive({
  name: "CPU1",
  value: 0,
  peak: 76,
  date: "",
});

const HIGH_COLOR = "orange";
const MEDIUM_COLOR = "gray";
const LOW_COLOR = "green";
const HIGH_COLOR_ACTIVE = "red";
const MEDIUM_COLOR_ACTIVE = "red";
const LOW_COLOR_ACTIVE = "red";
const barGraphPadding = 4;
const barGraphBorderSize = 1;
const oneBarHeight = 5;

const buildWrapper = () => {
  const wrapper = document.getElementById("chart-wrapper");
  wrapper.style.width = "77px";
  wrapper.style.height = "149px";
  wrapper.style.border = "solid 2px gray";
  wrapper.style.backgroundColor = "black";
  wrapper.style.position = "relative";
  wrapper.style.display = "flex";
  wrapper.style.flexDirection = "column";
};

const buildHeader = () => {
  const header = document.getElementById("chart-header");
  header.style.height = "40px";
};

const buildBody = () => {
  const body = document.getElementById("chart-body");
  body.style.height = "77px";

  buildBodyBarGraph();
};

const buildBodyBarGraph = () => {
  const barGraph = document.getElementById("bar-graph");
  barGraph.style.width = "37px";
  barGraph.style.height = "77px";
  barGraph.style.marginLeft = "14px";
  barGraph.style.padding = `${barGraphPadding}px 0`;
  barGraph.style.border = "solid 1px gray";

  const [levelDomId, activeColor] = getDomIdAndActiveColor();
  setBarItemUsageValue(levelDomId, activeColor);
};

const getDomIdAndActiveColor = () => {
  if (state.value <= 10) return ["low-active-1", LOW_COLOR_ACTIVE];
  if (state.value <= 20) return ["low-active-2", LOW_COLOR_ACTIVE];
  if (state.value <= 30) return ["low-active-3", LOW_COLOR_ACTIVE];
  if (state.value <= 40) return ["low-active-4", LOW_COLOR_ACTIVE];
  if (state.value <= 50) return ["low-active-5", LOW_COLOR_ACTIVE];
  if (state.value <= 60) return ["low-active-6", LOW_COLOR_ACTIVE];
  if (state.value <= 70) return ["medium-active-1", MEDIUM_COLOR_ACTIVE];
  if (state.value <= 80) return ["medium-active-2", MEDIUM_COLOR_ACTIVE];
  if (state.value <= 90) return ["medium-active-3", MEDIUM_COLOR_ACTIVE];
  if (state.value <= 10) return ["high-active", HIGH_COLOR_ACTIVE];
};

/**
 *
 * @param {*} id
 * @param {*} color
 * @param {*} value
 */
const setBarItemUsageValue = (id, color) => {
  // Set active color for bars whose value is less than the target bar
  const barNumber = Math.floor(state.value / 10);
  for (let i = 1; i <= 9; i++) {
    if (i <= barNumber) {
      const properties =
        i <= 6
          ? [`low-active-${i}`, LOW_COLOR_ACTIVE]
          : [`medium-active-${i - 6}`, MEDIUM_COLOR_ACTIVE];

      const lowLevelDom = document.getElementById(`${properties[0]}`);
      lowLevelDom.style.backgroundColor = properties[1];
      lowLevelDom.style.height = `${oneBarHeight}px`;
    }
  }

  // Compute and set active color for the target bar
  const computedHeight = (oneBarHeight / 10) * (state.value % 10);
  const targetBar = document.getElementById(id);
  targetBar.style.backgroundColor = color;
  targetBar.style.height = `${computedHeight}px`;
};

const buildPeakBox = () => {
  const borderBottomSize = 1;
  const peak = document.getElementById("peak-box");
  peak.style.bottom = `${
    barGraphPadding + barGraphBorderSize + borderBottomSize / 2
  }px`;
  peak.style.right = "0";
  peak.style.borderBottom = `solid ${borderBottomSize}px red`;
};

const renderInitialColors = () => {
  // High level color
  const high = document.getElementById("high");
  high.style.backgroundColor = `${HIGH_COLOR}`;
  // Medium level color
  const medium1 = document.getElementById("medium-1");
  const medium2 = document.getElementById("medium-2");
  const medium3 = document.getElementById("medium-3");
  medium1.style.backgroundColor = `${MEDIUM_COLOR}`;
  medium2.style.backgroundColor = `${MEDIUM_COLOR}`;
  medium3.style.backgroundColor = `${MEDIUM_COLOR}`;
  // Low level color
  const low1 = document.getElementById("low-1");
  const low2 = document.getElementById("low-2");
  const low3 = document.getElementById("low-3");
  const low4 = document.getElementById("low-4");
  const low5 = document.getElementById("low-5");
  const low6 = document.getElementById("low-6");
  low1.style.backgroundColor = `${LOW_COLOR}`;
  low2.style.backgroundColor = `${LOW_COLOR}`;
  low3.style.backgroundColor = `${LOW_COLOR}`;
  low4.style.backgroundColor = `${LOW_COLOR}`;
  low5.style.backgroundColor = `${LOW_COLOR}`;
  low6.style.backgroundColor = `${LOW_COLOR}`;
};

const renderGraph = () => {
  buildWrapper();
  buildHeader();
  buildBody();
  buildPeakBox();
  renderInitialColors();
};

onMounted(() => {
  renderGraph();

  const interval = setInterval(() => {
    state.value++;
    renderGraph();
    if (state.value === 67) {
      clearInterval(interval);
    }
  }, 10);
});
</script>

<template>
  <div id="chart-wrapper">
    <div id="chart-header">
      <div id="name">{{ state.name }}</div>
    </div>
    <div id="chart-body">
      <div id="bar-graph">
        <div class="bar" id="high">
          <div class="bar-active" id="high-active"></div>
        </div>
        <div class="bar" id="medium-3">
          <div class="bar-active" id="medium-active-3"></div>
        </div>
        <div class="bar" id="medium-2">
          <div class="bar-active" id="medium-active-2"></div>
        </div>
        <div class="bar" id="medium-1">
          <div class="bar-active" id="medium-active-1"></div>
        </div>
        <div class="bar" id="low-6">
          <div class="bar-active" id="low-active-6"></div>
        </div>
        <div class="bar" id="low-5">
          <div class="bar-active" id="low-active-5"></div>
        </div>
        <div class="bar" id="low-4">
          <div class="bar-active" id="low-active-4"></div>
        </div>
        <div class="bar" id="low-3">
          <div class="bar-active" id="low-active-3"></div>
        </div>
        <div class="bar" id="low-2">
          <div class="bar-active" id="low-active-2"></div>
        </div>
        <div class="bar" id="low-1">
          <div class="bar-active" id="low-active-1"></div>
        </div>
      </div>
      <div id="peak-box">
        <div id="peak-value">
          {{ state.peak + "%" }}
        </div>
      </div>
    </div>
    <div id="chart-bottom">
      <div id="current-value">{{ state.value + "%" }}</div>
    </div>
  </div>
</template>

<style scoped lang="scss">
* {
  box-sizing: border-box !important;
  font-size: 10px;
  transition: 0.1s;
}

#chart-wrapper {
  #chart-header {
    width: 100%;
    display: flex;
    align-items: center;

    #name {
      width: 100%;
      text-align: center;
    }
  }

  #chart-body {
    width: 100%;
    position: relative;

    #bar-graph {
      display: flex;
      flex-direction: column;
      gap: 2px;

      .bar {
        width: 27px;
        height: 5px;
        overflow: hidden;
        margin: auto;
        position: relative;

        .bar-active {
          width: 100%;
          margin: auto;
          position: absolute;
          bottom: 0;
        }
      }
    }

    #peak-box {
      position: absolute;
      box-sizing: border-box;
    }
  }

  #chart-bottom {
    width: 100%;
    display: flex;
    align-items: center;

    #current-value {
      width: 100%;
      text-align: center;
    }
  }
}
</style>
