<script setup lang="ts">
import { computed, onMounted, reactive, watch } from "vue";

/**
 * @private
 */
type Props = {
  name: string;
  currentValue: number;
  peakValue: number;
  recordedDateTime: string;
};

type BarAlias =
  | "LOW_1"
  | "LOW_2"
  | "LOW_3"
  | "LOW_4"
  | "LOW_5"
  | "LOW_6"
  | "MEDIUM_1"
  | "MEDIUM_2"
  | "MEDIUM_3"
  | "HIGH";

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

const buildWrapper = () => {
  const wrapper = document.getElementById("graph-wrapper");
  wrapper.style.width = "77px";
  wrapper.style.height = "149px";
  wrapper.style.border = "solid 2px gray";
  wrapper.style.backgroundColor = "black";
};

const buildHeader = () => {
  const header = document.getElementById("graph-header");
  header.style.height = "40px";
  header.style.color = "white";
};

const buildBottom = () => {
  const bottom = document.getElementById("graph-bottom");
  bottom.style.height = "29px";
};

const buildBody = () => {
  const body = document.getElementById("graph-body");
  body.style.height = `${BAR_GRAPH_HEIGHT}px`;

  buildBodyBarGraph();
};

const buildBodyBarGraph = () => {
  const barGraph = document.getElementById("bar-graph");
  barGraph.style.width = "37px";
  barGraph.style.height = "77px";
  barGraph.style.marginLeft = "14px";
  barGraph.style.padding = `${BAR_GRAPH_PADDING}px 0`;
  barGraph.style.border = "solid 1px gray";
};

/**
 * @private
 */
const buildPeakBox = () => {
  const peakBox = document.getElementById("peak-box");
  if (peakBox) {
    const initBottom =
      BAR_GRAPH_PADDING + BAR_GRAPH_BORDER_SIZE + PEAK_BOTTOM_LINE_SIZE / 2;
    peakBox.style.bottom = `${initBottom}px`;
    peakBox.style.right = "1px";
    peakBox.style.width = "20px";
  }
};

/**
 * @private
 */
const calculateTotalGapSize = () => {
  if (state.peak <= 10) return 0;
  if (state.peak % 10 === 0) return (state.peak / 10 - 1) * BAR_GRAPH_GAP_SIZE;
  return Math.floor(state.peak / 10) * BAR_GRAPH_GAP_SIZE;
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
      // setGraphColors();
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
      // buildPeakBox();
      if (state.peak === newValue) {
        original.peak = state.peak;
        clearInterval(interval);
      }
    }, 10);
  }
);

const computedPeakPosition = computed(() => {
  const onePercentHeight = ONE_BAR_HEIGHT / 10;
  const totalGapSize = calculateTotalGapSize();
  const peakPosition = onePercentHeight * state.peak + totalGapSize;
  return peakPosition;
});

const computedPeakColor = computed(() => {
  if (state.peak <= 60) return LOW_COLOR_ACTIVE;
  if (state.peak <= 90) return MEDIUM_COLOR_ACTIVE;
  if (state.peak <= 100) return HIGH_COLOR_ACTIVE;
});

/**
 * @private
 * @param alias: BarAlias
 */
const calculateBarHeight = (alias: BarAlias) => {
  const arr = [
    "LOW_1",
    "LOW_2",
    "LOW_3",
    "LOW_4",
    "LOW_5",
    "LOW_6",
    "MEDIUM_1",
    "MEDIUM_2",
    "MEDIUM_3",
    "HIGH",
  ];

  for (let i = 0; i < arr.length; i++) {
    if (arr[i] === alias) {
      if (state.value >= (i + 1) * 10) return ONE_BAR_HEIGHT;
      if (state.value > i * 10)
        return (ONE_BAR_HEIGHT / 10) * (state.value % 10);
      return 0;
    }
  }
};

const computedActiveBarLow1 = computed(() => {
  return calculateBarHeight("LOW_1");
});

const computedActiveBarLow2 = computed(() => {
  return calculateBarHeight("LOW_2");
});

const computedActiveBarLow3 = computed(() => {
  return calculateBarHeight("LOW_3");
});

const computedActiveBarLow4 = computed(() => {
  return calculateBarHeight("LOW_4");
});

const computedActiveBarLow5 = computed(() => {
  return calculateBarHeight("LOW_5");
});

const computedActiveBarLow6 = computed(() => {
  return calculateBarHeight("LOW_6");
});

const computedActiveBarMedium1 = computed(() => {
  return calculateBarHeight("MEDIUM_1");
});

const computedActiveBarMedium2 = computed(() => {
  return calculateBarHeight("MEDIUM_2");
});

const computedActiveBarMedium3 = computed(() => {
  return calculateBarHeight("MEDIUM_3");
});

const computedActiveBarHigh = computed(() => {
  return calculateBarHeight("HIGH");
});

/**
 * @private
 */
onMounted(() => {
  renderGraph();
});
</script>

<template>
  <div class="graph-wrapper" id="graph-wrapper">
    <div class="graph-header" id="graph-header">
      <div class="name">{{ state.name }}</div>
    </div>
    <div class="graph-body" id="graph-body">
      <div class="bar-graph" id="bar-graph">
        <div class="bar" id="high">
          <div
            class="bar-active high-active"
            :style="{ height: computedActiveBarHigh + 'px' }"
          ></div>
        </div>
        <div class="bar" id="medium-3">
          <div
            class="bar-active medium-active"
            :style="{ height: computedActiveBarMedium3 + 'px' }"
          ></div>
        </div>
        <div class="bar" id="medium-2">
          <div
            class="bar-active medium-active"
            :style="{ height: computedActiveBarMedium2 + 'px' }"
          ></div>
        </div>
        <div class="bar" id="medium-1">
          <div
            class="bar-active medium-active"
            :style="{ height: computedActiveBarMedium1 + 'px' }"
          ></div>
        </div>
        <div class="bar" id="low-6">
          <div
            class="bar-active low-active"
            :style="{ height: computedActiveBarLow6 + 'px' }"
          ></div>
        </div>
        <div class="bar" id="low-5">
          <div
            class="bar-active low-active"
            :style="{ height: computedActiveBarLow5 + 'px' }"
          ></div>
        </div>
        <div class="bar" id="low-4">
          <div
            class="bar-active low-active"
            :style="{ height: computedActiveBarLow4 + 'px' }"
          ></div>
        </div>
        <div class="bar" id="low-3">
          <div
            class="bar-active low-active"
            :style="{ height: computedActiveBarLow3 + 'px' }"
          ></div>
        </div>
        <div class="bar" id="low-2">
          <div
            class="bar-active low-active"
            :style="{ height: computedActiveBarLow2 + 'px' }"
          ></div>
        </div>
        <div class="bar" id="low-1">
          <div
            class="bar-active low-active"
            :style="{ height: computedActiveBarLow1 + 'px' }"
          ></div>
        </div>
      </div>
      <div
        id="peak-box"
        :style="{
          transform: `translateY(-${computedPeakPosition}px)`,
          color: computedPeakColor,
          borderBottom: `solid ${PEAK_BOTTOM_LINE_SIZE}px ${computedPeakColor}`,
        }"
      >
        <div id="peak-value">
          {{ state.peak + "%" }}
        </div>
        <div
          id="triangle"
          :style="{
            borderLeft: `${TRIANGLE_SIZE - 1}px solid transparent`,
            borderRight: `${TRIANGLE_SIZE - 1}px solid transparent`,
            borderTop: `${TRIANGLE_SIZE}px solid ${computedPeakColor}`,
          }"
        ></div>
      </div>
    </div>
    <div class="graph-bottom" id="graph-bottom">
      <div class="current-value">{{ state.value + "%" }}</div>
    </div>
  </div>
</template>

<style scoped lang="scss">
* {
  box-sizing: border-box !important;
  font-size: 8px;
  transition: 0.1s;
}

.low-active {
  background-color: green;
}
.medium-active {
  background-color: blue;
}

.high-active {
  background-color: gold;
}

.graph-wrapper {
  .graph-header {
    width: 100%;
    margin: auto;
    display: flex;
    padding: 0 22px 0 14px;
    align-items: center;

    .name {
      width: 100%;
      text-align: center;
    }
  }

  .graph-body {
    width: 100%;
    position: relative;

    .bar-graph {
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

    .peak-box {
      position: absolute;
      text-align: center;

      .triangle {
        width: 0;
        height: 0;
        margin: auto;
      }
    }
  }

  .graph-bottom {
    width: 100%;
    display: flex;
    align-items: center;
    padding: 0 22px 0 14px;

    .current-value {
      width: 100%;
      text-align: center;
    }
  }
}
</style>
