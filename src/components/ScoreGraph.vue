<template>
    <h4 class="chart-title">Évolution de la partie</h4>
    <div class="chart-container" v-if="gameData">
        <!-- Affiche les joueurs avec leur nom et score -->
        <div class="players-container">
            <div
                v-for="player in sortedPlayers"
                :key="player.name"
                class="player-item"
                :style="{ color: player.color }"
            >
                <span class="player-name-text">{{ player.name }}</span>
                <span class="player-score">({{ player.score }}pt)</span>
            </div>
        </div>
        <v-chart :option="chartOptions" autoresize></v-chart>
    </div>
</template>


<script setup>
import { ref, watch, defineProps, computed, onMounted } from "vue";
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

const sortedPlayers = computed(() => {
    return props.players
        .map((player, index) => {
            // Récupère la dernière valeur de l'array 'score' pour obtenir le score total
            const scores = props.gameData.data[player].score;
            const totalScore = scores[scores.length - 1] || 0; // Si le tableau est vide, retourne 0
            return { name: player, color: props.playerColors[index], score: totalScore };
        })
        .sort((a, b) => b.score - a.score); // Trie par score décroissant
});


const chartOptions = ref({
    tooltip: {
        trigger: "axis",
        backgroundColor: "rgba(50, 50, 50, 0.7)",
        borderRadius: 5,
        textStyle: {
            color: "#fff",
        },
        formatter: (params) => {
            // Trie les séries par valeur décroissante
            const sortedParams = params.sort((a, b) => b.data - a.data);

            // Construire le contenu HTML du tooltip trié
            let tooltipContent = `${params[0].axisValue}<br/>`; // Affiche "T5" ou autre valeur sur l'axe des X
            sortedParams.forEach((item) => {
                tooltipContent += `
        <span style="display:inline-block;margin-right:5px;border-radius:10px;width:10px;height:10px;background-color:${item.color};"></span>
        ${item.seriesName}: ${item.data}<br/>
      `;
            });

            return tooltipContent;
        },
    },
    grid: {
        top: "10%",
        left: "5%",
        right: "15%",
        bottom: "10%",
        containLabel: true,
    },
    xAxis: {
        type: "category",
        boundaryGap: false,
        data: [],
        nameTextStyle: {
            fontWeight: "bold",
        },
        axisLine: {
            lineStyle: {
                color: "#aaa",
            },
        },
        axisLabel: {
            color: "#555",
        },
    },
    yAxis: {
        type: "value",
        nameTextStyle: {
            fontWeight: "bold",
        },
        axisLine: {
            lineStyle: {
                color: "#aaa",
            },
        },
        axisLabel: {
            color: "#555",
        },
        splitLine: {
            lineStyle: {
                type: "solid",
                color: "#ddd",
            },
        },
    },
    series: [],
});

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
                width: 3,
            },
            smooth: false,
            label: {
                show: true,
                position: "right",
                formatter: (params) => {
                    return params.dataIndex === props.gameData.data[player].score.length - 1
                        ? `{color|${params.value}}`
                        : "";
                },
                rich: {
                    color: {
                        color: props.playerColors[index],
                        fontWeight: "bold",
                        fontSize: 12,
                    },
                },
            },
            symbol: "circle",
            symbolSize: 8,
            itemStyle: {
                color: props.playerColors[index],
            },
        })),
    };
};

onMounted(() => {
    if (props.gameData) {
        updateChart(); // Charge le graphique lors de la montée du composant
    }
});

watch(
    () => props.gameData,
    updateChart,
    { deep: true }
);

</script>

<style scoped>

.chart-title {
    font-size: clamp(0px, 15px, 3vw);
    font-weight: 400;
    text-align: center;
    color: #5F5F5F;
    margin-top: clamp(0px, 10px, 2vw);
}

.chart-container {
    width: 100%;
    height: clamp(0px, 350px, 70vw);
    padding-bottom: clamp(0px, 50px, 10vw);
    border-radius: clamp(0px, 10px, 2vw);
    border: 0px solid #ccc;
    box-shadow: 0 clamp(0px, 5px, 1vw) clamp(0px, 10px, 2vw) rgba(0, 0, 0, 0.123);
}


.players-container {
    display: inline-flex;
    flex-wrap: wrap; /* Permet de passer à la ligne si nécessaire */
    justify-content: space-evenly; /* Répartition uniforme */
    width: 100%; /* Prend toute la largeur disponible */
    text-align: center;
    margin-top: clamp(0px, 20px, 4vw);
}

.player-item {
    display: flex;
    flex-direction: column;
    align-items: center;
    min-width: fit-content; /* Ajuste la largeur au contenu */
}

.player-name-text {
    font-weight: 600;
    font-size: clamp(0px, 20px, 4vw);
}

.player-score {
    font-weight: 500;
    font-size: clamp(0px, 17.5px, 3.5vw);
}

</style>