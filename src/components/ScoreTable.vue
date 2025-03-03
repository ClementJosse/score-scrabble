<template>
    <h4 class="table-title">Scores par tour</h4>
    <div class="score-container">
        <div class="tables-wrapper">

            <div class="background-columns">
                <div class="background-light-column"></div>
                <div class="background-dark-column"></div>
                <div class="background-light-column"></div>
                <div class="background-dark-column"></div>
                <div class="background-light-column"></div>
                <div class="background-dark-column"></div>
                <div class="background-light-column"></div>
                <div class="background-dark-column"></div>
            </div>
            <!-- Tableau des joueurs -->
            <table class="player-table">
                <thead>
                    <tr>
                        <th class="tour">Tour</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="player in players" :key="player">
                        <div class="player-row">
                            <div class="turn-rectangle" :style="{ backgroundColor: isPLayerTurn(player) }"></div>
                            <span class="playername-table" :style="{ color: getPlayerColor(player) }">{{ player
                                }}</span>
                        </div>
                    </tr>
                </tbody>
            </table>

            <!-- Tableau des scores -->
            <div ref="scoreContainer" class="scores-wrapper">
                <table class="score-table">
                    <thead>
                        <tr>
                            <th v-for="index in maxTurns" :key="'header-' + index"
                                :style="{ backgroundColor: index % 2 === 0 ? '#FDFDFD' : '#F8F8F8' }">
                                <span v-if="isColumnFilled(index)">T{{ index }}</span>
                            </th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="player in players" :key="player">
                            <td v-for="index in maxTurns" :key="'score-' + player + '-' + index"
                                :style="{ backgroundColor: index % 2 === 0 ? '#FDFDFD' : '#F8F8F8', width: 'clamp(0px, 35.3125px, 7.0625vw)' }">
                                <span
                                    v-if="getPlayedForPlayer(player)[index - 1] !== undefined && getPlayedForPlayer(player)[index - 1] !== '-'"
                                    :style="{ color: getScoreColor(getPlayedForPlayer(player)[index - 1]) }">
                                    {{ getPlayedForPlayer(player)[index - 1] }}
                                </span>
                                <span v-else>.</span>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, nextTick, onMounted } from 'vue';
import { defineProps } from 'vue';

const props = defineProps({
    players: Array,
    gameData: Object,
    maxTurns: Number,
    currentPlayer: String,
    playerColors: Array,
});

const scoreContainer = ref(null);

const getPlayedForPlayer = (player) => {
    return props.gameData.data[player].played.length > 0
        ? props.gameData.data[player].played
        : [];
};

const getPlayerColor = (player) => {
    const playerIndex = props.players.indexOf(player);
    return props.playerColors[playerIndex] || '#000';
};

const getScoreColor = (value) => {
    scrollToRight();
    if (value > 0) return 'rgb(0, 75, 53)'; // Vert
    if (value <= 0) return 'rgb(75, 0, 1)'; // Rouge
    return 'rgb(253, 253, 253)'; // Par défaut
};

const isPLayerTurn = (player) => {
    return props.currentPlayer === player ? getPlayerColor(player) : '#ffffff';
};

const isColumnFilled = (columnIndex) => {
    return props.players.some((player) => {
        return props.gameData.data[player].played[columnIndex - 1] !== undefined;
    });
};

const scrollToRight = () => {
    if (scoreContainer.value) {
        scoreContainer.value.scrollLeft = scoreContainer.value.scrollWidth;
    }
};

onMounted(async () => {
    try {
        nextTick(() => {
            scrollToRight();
        });
    } catch (e) {
        console.error('Erreur lors du chargement du fichier JSON :', e);
    }
});
</script>

<style scoped>
.table-title {
    font-size: clamp(0px, 15px, 3vw);
    font-weight: 400;
    text-align: center;
    color: #5F5F5F;
    padding-top: clamp(0px, 40px, 8vw);
}

.score-container {
    overflow-x: auto;
    /* Ajoute un défilement horizontal */
    white-space: nowrap;
    /* Empêche les colonnes de se replier */
    border-radius: clamp(0px, 10px, 2vw);
    border: 0px solid #ccc;
    box-shadow: 0 clamp(0px, 5px, 1vw) clamp(0px, 10px, 2vw) rgba(0, 0, 0, 0.123);
}

.score-table {
    border-collapse: collapse;
    width: max-content;
    /* Ajuste la largeur selon le contenu */
}

.tables-wrapper {
    position: relative; /* Position relative pour que les éléments absolus soient alignés à l'intérieur */
    display: flex;
    align-items: flex-start;
}

.player-table {
    border-collapse: collapse;
    min-width: clamp(0px, 100px, 20vw);
    border: 0px;
    margin-right: clamp(0px, 5px, 1vw);
    overflow: hidden;
    background-color: #ffffff;
}

.player-table th,
.player-table td {
    text-align: right;
    white-space: nowrap;
    /* Empêche le texte de se couper */
    background-color: #ffffff;
    border: 0px;
}

.player-row {
    display: flex;
    height: clamp(0px, 45px, 9vw);
    align-items: center;
}

.playername-table {
    font-size: clamp(0px, 20px, 4vw);
    font-weight: 600;
    text-overflow: ellipsis;
    overflow: hidden;
    max-width: clamp(0px, 100px, 20vw);
    min-width: clamp(0px, 100px, 20vw);
    text-align: right;
}

.turn-rectangle {
    display: flex;
    width: clamp(0px, 7.5px, 1.5vw);
    height: clamp(0px, 35px, 7vw);
    border-bottom: clamp(0px, 15px, 3vw);
    margin-right: clamp(0px, 5px, 1vw)
}

.scores-wrapper {
    overflow-x: auto;
    white-space: nowrap;
}

.score-table {
    border-collapse: collapse;
    width: max-content;
    /* Ajuste la largeur selon le contenu */
}

.score-table th,
.score-table td {
    min-width: clamp(0px, 35.3125px, 7.0625vw);
    text-align: center;
    border: 0px;
    white-space: nowrap;
    /* Empêche le texte de se couper */
}

.score-table th {
    font-size: clamp(0px, 15px, 3vw);
}

.score-table td {
    font-size: clamp(0px, 20px, 4vw);
}

.score-table {
    background-color: #ffffff;
    border-radius: clamp(0px, 10px, 2vw);
    border: 0px solid #ccc;
    box-shadow: 0 clamp(0px, 5px, 1vw) clamp(0px, 10px, 2vw) rgba(0, 0, 0, 0.123);
    width: 100%;
}

.tour {
    display: flex;
    color: #5F5F5F;
    justify-content: center;
    align-items: center;
    font-size: clamp(0px, 17.5px, 3.5vw);
}

th {
    height: clamp(0px, 40px, 8vw);
}

td {
    height: clamp(0px, 45px, 9vw);
}

th {
    color: #5F5F5F;
    font-weight: 500;
}

td {
    font-weight: 600;
}

.background-columns {
    position: absolute;
    top: 0;
    right: 0; /* Commencer depuis la droite */
    height: 100%;
    display: flex;
    flex-direction: row-reverse; /* Commence par la droite pour que les colonnes soient empilées dans l'ordre inverse */
    z-index: -1; /* Assurez-vous que c'est à l'arrière-plan */
}

.background-light-column,
.background-dark-column {
    width: clamp(0px, 35.3125px, 7.0625vw);
    height: 100%;
}

.background-light-column {
    background-color: rgb(253, 253, 253);
}

.background-dark-column {
    background-color: rgb(248, 248, 248);
}
</style>