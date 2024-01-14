<script setup lang="ts">
import { onMounted, reactive, watch } from "vue";

/**
 * @private
 */
type Props = {
  name: string;
  currentValue: number;
  peakValue: number;
  recordedDateTime: string;
};

/**
 * @private
 */
const props = withDefaults(defineProps<Props>(), {
  name: () => "",
  currentValue: () => 0,
  peakValue: () => 0,
  recordedDateTime: () => "",
});

/**
 * @private
 */
const state = reactive({
  name: props.name,
  value: props.currentValue,
  peak: props.peakValue,
  date: props.recordedDateTime,
});

/**
 * @private
 */
const original = {
  ...state,
};
const HIGH_COLOR = "#b31102";
const HIGH_COLOR_ACTIVE = "#f62b04";
const MEDIUM_COLOR = "#433f17";
const MEDIUM_COLOR_ACTIVE = "#fbd453";
const LOW_COLOR = "#47067e";
const LOW_COLOR_ACTIVE = "#8470f8";
const BAR_GRAPH_HEIGHT = 77;
const BAR_GRAPH_PADDING = 4;
const BAR_GRAPH_BORDER_SIZE = 1;
const BAR_GRAPH_GAP_SIZE = 2;
const ONE_BAR_HEIGHT = 5;
const PEAK_BOTTOM_LINE_SIZE = 1;
const TRIANGLE_SIZE = 4;

/**
 * @private
 */
const buildWrapper = () => {
  const wrapper = document.getElementById("graph-wrapper");
  wrapper.style.width = "77px";
  wrapper.style.height = "149px";
  wrapper.style.border = "solid 2px gray";
  wrapper.style.backgroundColor = "black";
};

/**
 * @private
 */
const buildHeader = () => {
  const header = document.getElementById("graph-header");
  header.style.height = "40px";
  header.style.color = "white";
};

/**
 * @private
 */
const buildBottom = () => {
  const bottom = document.getElementById("graph-bottom");
  bottom.style.height = "29px";
};

/**
 * @private
 */
const buildBody = () => {
  const body = document.getElementById("graph-body");
  body.style.height = `${BAR_GRAPH_HEIGHT}px`;

  buildBodyBarGraph();
};

/**
 * @private
 */
const buildBodyBarGraph = () => {
  const barGraph = document.getElementById("bar-graph");
  barGraph.style.width = "37px";
  barGraph.style.height = "77px";
  barGraph.style.marginLeft = "14px";
  barGraph.style.padding = `${BAR_GRAPH_PADDING}px 0`;
  barGraph.style.border = "solid 1px gray";

  setGraphColors();
};

/**
 * @private
 */
const getPropertiesForTargetBar = () => {
  if (state.value < 0 || state.value > 100) return [];
  if (state.value <= 10) return ["low-active-1", LOW_COLOR_ACTIVE];
  if (state.value <= 20) return ["low-active-2", LOW_COLOR_ACTIVE];
  if (state.value <= 30) return ["low-active-3", LOW_COLOR_ACTIVE];
  if (state.value <= 40) return ["low-active-4", LOW_COLOR_ACTIVE];
  if (state.value <= 50) return ["low-active-5", LOW_COLOR_ACTIVE];
  if (state.value <= 60) return ["low-active-6", LOW_COLOR_ACTIVE];
  if (state.value <= 70) return ["medium-active-1", MEDIUM_COLOR_ACTIVE];
  if (state.value <= 80) return ["medium-active-2", MEDIUM_COLOR_ACTIVE];
  if (state.value <= 90) return ["medium-active-3", MEDIUM_COLOR_ACTIVE];
  if (state.value <= 100) return ["high-active", HIGH_COLOR_ACTIVE];
};

/**
 * @private
 */
const setGraphColors = () => {
  // Set active color for bars whose value is less than the target bar
  const barNumber = Math.floor(state.value / 10);
  for (let i = 1; i <= 10; i++) {
    if (i <= barNumber) {
      const properties =
        i <= 6
          ? [`low-active-${i}`, LOW_COLOR_ACTIVE]
          : [`medium-active-${i - 6}`, MEDIUM_COLOR_ACTIVE];

      const barItemDom = document.getElementById(`${properties[0]}`);
      if (barItemDom) {
        barItemDom.style.backgroundColor = properties[1];
        barItemDom.style.height = `${ONE_BAR_HEIGHT}px`;
      }
    } else if (i > barNumber && i < 10) {
      // Clear history
      const properties =
        i <= 6
          ? [`low-active-${i}`, LOW_COLOR_ACTIVE]
          : [`medium-active-${i - 6}`, MEDIUM_COLOR_ACTIVE];

      const barItemDom = document.getElementById(`${properties[0]}`);
      barItemDom!.style.height = "0";
    } else {
      // Clear history
      const highLevelDom = document.getElementById(`high-active`);
      highLevelDom!.style.height = "0";
    }
  }

  // Compute and set active color for the target bar
  if (state.value % 10 !== 0) {
    const [levelDomId, activeColor] = getPropertiesForTargetBar();
    const computedHeight = (ONE_BAR_HEIGHT / 10) * (state.value % 10);
    const targetBar = document.getElementById(levelDomId);
    if (targetBar) {
      targetBar.style.backgroundColor = activeColor;
      targetBar.style.height = `${computedHeight}px`;
    }
  }

  // Set current percent value colors
  const currentValue = document.getElementById("current-value");
  currentValue!.style.color = computePercentValueColor(state.value);
};

/**
 * @private
 */
const buildPeakBox = () => {
  const initBottom =
    BAR_GRAPH_PADDING + BAR_GRAPH_BORDER_SIZE + PEAK_BOTTOM_LINE_SIZE / 2;
  const onePercentHeight = ONE_BAR_HEIGHT / 10;
  const totalGapSize = computeTotalGapSize();
  const peakPosition = onePercentHeight * state.peak + totalGapSize;

  const peakBox = document.getElementById("peak-box");
  if (peakBox) {
    const peakColor = computePercentValueColor(state.peak);

    peakBox.style.color = peakColor;
    peakBox.style.borderBottom = `solid ${PEAK_BOTTOM_LINE_SIZE}px ${peakColor}`;
    peakBox.style.right = "1px";
    peakBox.style.width = "20px";
    peakBox.style.bottom = `${initBottom}px`;
    peakBox.style.transform = `translateY(-${peakPosition}px)`;

    const triangleIcon = document.getElementById("triangle");
    if (triangleIcon) {
      triangleIcon.style.borderLeft = `${
        TRIANGLE_SIZE - 1
      }px solid transparent`;
      triangleIcon.style.borderRight = `${
        TRIANGLE_SIZE - 1
      }px solid transparent`;
      triangleIcon.style.borderTop = `${TRIANGLE_SIZE}px solid ${peakColor}`;
    }
  }
};

/**
 * @private
 */
const computeTotalGapSize = () => {
  if (state.peak <= 10) return 0;
  if (state.peak % 10 === 0) return (state.peak / 10 - 1) * BAR_GRAPH_GAP_SIZE;
  return Math.floor(state.peak / 10) * BAR_GRAPH_GAP_SIZE;
};

/**
 * @private
 */
const computePercentValueColor = (value: number) => {
  if (value <= 60) return LOW_COLOR_ACTIVE;
  if (value <= 90) return MEDIUM_COLOR_ACTIVE;
  if (value <= 100) return HIGH_COLOR_ACTIVE;
};

/**
 * @private
 */
const setInitBarColors = () => {
  // High level color
  const high = document.getElementById("high");
  high.style.backgroundColor = `${HIGH_COLOR}`;

  // Medium level color
  for (let i = 1; i <= 3; i++) {
    const mediumDom = document.getElementById(`medium-${i}`);
    mediumDom.style.backgroundColor = `${MEDIUM_COLOR}`;
  }

  // Low level color
  for (let i = 1; i <= 6; i++) {
    const lowDom = document.getElementById(`low-${i}`);
    lowDom.style.backgroundColor = `${LOW_COLOR}`;
  }
};

/**
 * @private
 */
const renderGraph = () => {
  buildWrapper();
  buildHeader();
  buildBottom();
  buildBody();
  setInitBarColors();
  buildPeakBox();
};

/**
 * @private
 */
watch(
  () => props.currentValue,
  (newValue) => {
    if (newValue < 0 || newValue > 100 || newValue === original.value) {
      return;
    }

    const interval = setInterval(() => {
      if (original.value < newValue) {
        state.value++;
      } else {
        state.value--;
      }
      // Update the bar graph
      setGraphColors();
      if (state.value === newValue) {
        original.value = state.value;
        clearInterval(interval);
      }
    }, 0);
  }
);

/**
 * @private
 */
watch(
  () => props.name,
  (newValue) => {
    state.name = newValue;
  }
);

/**
 * @private
 */
watch(
  () => props.peakValue,
  (newValue) => {
    if (newValue < 0 || newValue > 100 || newValue === original.peakValue) {
      return;
    }

    const interval = setInterval(() => {
      if (original.peak < newValue) {
        state.peak++;
      } else {
        state.peak--;
      }
      // Update the peak box
      buildPeakBox();
      if (state.peak === newValue) {
        original.peak = state.peak;
        clearInterval(interval);
      }
    }, 0);
  }
);

/**
 * @private
 */
onMounted(() => {
  renderGraph();
});
</script>

<template>
  <div id="graph-wrapper">
    <div id="graph-header">
      <div id="name">{{ state.name }}</div>
    </div>
    <div id="graph-body">
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
        <div id="triangle"></div>
      </div>
    </div>
    <div id="graph-bottom">
      <div id="current-value">{{ state.value + "%" }}</div>
    </div>
  </div>
</template>

<style scoped lang="scss">
* {
  box-sizing: border-box !important;
  font-size: 8px;
  transition: 0.1s;
}

#graph-wrapper {
  #graph-header {
    width: 100%;
    margin: auto;
    display: flex;
    padding: 0 22px 0 14px;
    align-items: center;

    #name {
      width: 100%;
      text-align: center;
    }
  }

  #graph-body {
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
      text-align: center;

      #triangle {
        width: 0;
        height: 0;
        margin: auto;
      }
    }
  }

  #graph-bottom {
    width: 100%;
    display: flex;
    align-items: center;
    padding: 0 22px 0 14px;

    #current-value {
      width: 100%;
      text-align: center;
    }
  }
}
</style>
