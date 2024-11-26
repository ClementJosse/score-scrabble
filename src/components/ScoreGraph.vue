<template>
    <div class="chart-container" v-if="gameData">
      <h4 class="chart-title">Statistiques</h4>
      <v-chart :option="chartOptions" autoresize></v-chart>
    </div>
  </template>
  
  <script setup>
  import { ref, watch, defineProps} from "vue";
  import { use } from "echarts/core";
  import { CanvasRenderer } from "echarts/renderers";
  import { LineChart } from "echarts/charts";
  import { GridComponent, TooltipComponent, LegendComponent } from "echarts/components";
  import VChart from "vue-echarts";
  
  // Configure ECharts
  use([CanvasRenderer, LineChart, GridComponent, TooltipComponent, LegendComponent]);
  
  const props = defineProps({
    gameData: Object,
    players: Array,
    playerColors: Array,
  });
  
  const chartOptions = ref({
    tooltip: {
      trigger: "axis",
    },
    legend: {
      show: true, // Générée automatiquement
    },
    grid: {
      left: "3%",
      right: "4%",
      bottom: "3%",
      containLabel: true,
    },
    xAxis: {
      type: "category",
      data: [],
      name: "Tour",
    },
    yAxis: {
      type: "value",
      name: "Score",
    },
    series: [],
  });
  
  // Mettre à jour les options dynamiquement
  const updateChart = () => {
    const maxTurns = Math.max(...props.players.map(player => props.gameData.data[player].score.length));
    const labels = Array.from({ length: maxTurns }, (_, i) => `T${i + 1}`);
  
    chartOptions.value = {
      ...chartOptions.value,
      xAxis: { ...chartOptions.value.xAxis, data: labels },
      series: props.players.map((player, index) => ({
        name: player,
        type: "line",
        data: props.gameData.data[player].score,
        lineStyle: {
          color: props.playerColors[index],
        },
      })),
    };
  };
  
  // Mettre à jour le graphique lorsque les données changent
  watch(
    () => props.gameData,
    updateChart,
    { deep: true }
  );
  </script>
  
  <style scoped>
  .chart-container {
    width: 100%;
    height: 400px;
  }
  
  .chart-title {
    font-size: 16px;
    font-weight: bold;
    text-align: center;
    margin-bottom: 10px;
  }
  </style>
  