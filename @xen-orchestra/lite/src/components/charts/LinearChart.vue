<template>
  <UiCard class="linear-chart">
    <VueCharts :option="option" autoresize class="chart" />
    <slot name="summary" />
  </UiCard>
</template>

<script lang="ts" setup>
import UiCard from "@/components/ui/UiCard.vue";
import type { LinearChartData, ValueFormatter } from "@/types/chart";
import { IK_CHART_VALUE_FORMATTER } from "@/types/injection-keys";
import { utcFormat } from "d3-time-format";
import type { EChartsOption } from "echarts";
import { LineChart } from "echarts/charts";
import {
  GridComponent,
  LegendComponent,
  TitleComponent,
  TooltipComponent,
} from "echarts/components";
import { use } from "echarts/core";
import { CanvasRenderer } from "echarts/renderers";
import { computed, provide } from "vue";
import VueCharts from "vue-echarts";

const Y_AXIS_MAX_VALUE = 200;

const props = defineProps<{
  title?: string;
  subtitle?: string;
  data: LinearChartData;
  valueFormatter?: ValueFormatter;
  maxValue?: number;
}>();

const valueFormatter = computed<ValueFormatter>(() => {
  const formatter = props.valueFormatter;

  return (value) => {
    if (formatter === undefined) {
      return value.toString();
    }

    return formatter(value);
  };
});

provide(IK_CHART_VALUE_FORMATTER, valueFormatter);

use([
  CanvasRenderer,
  LineChart,
  GridComponent,
  TooltipComponent,
  TitleComponent,
  LegendComponent,
]);

const option = computed<EChartsOption>(() => ({
  title: {
    text: props.title,
    subtext: props.subtitle,
  },
  legend: {
    data: props.data.map((series) => series.label),
  },
  tooltip: {
    valueFormatter: (v) => valueFormatter.value(v as number),
  },
  xAxis: {
    type: "time",
    axisLabel: {
      formatter: (timestamp: number) =>
        utcFormat("%a\n%I:%M\n%p")(new Date(timestamp)),
      showMaxLabel: false,
      showMinLabel: false,
    },
  },
  yAxis: {
    type: "value",
    axisLabel: {
      formatter: valueFormatter.value,
    },
    max: props.maxValue ?? Y_AXIS_MAX_VALUE,
  },
  series: props.data.map((series, index) => ({
    type: "line",
    name: series.label,
    zlevel: index + 1,
    data: series.data.map((item) => [item.timestamp, item.value]),
  })),
}));
</script>

<style lang="postcss" scoped>
.chart {
  width: 100%;
  height: 30rem;
}
</style>
