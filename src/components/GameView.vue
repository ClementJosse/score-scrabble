<template>
    <div class="back-container">
      <button @click="$emit('goBack')" class="back-menu">Retour</button>
    </div>
    <div v-if="gameData" class="gameData">
      <h2>Au tour de:</h2>
      <h1 :style="{ color: currentPlayerColor }">{{ gameData['current-turn'].toUpperCase() }}</h1>
  
      
      <div class="input-move">
        <h4>Score:</h4>
        <input v-model="newMove" type="number" placeholder="Ajouter un coup" />
        <button @click="addMove">Ajouter</button>
      </div>
  
      <!-- Tableau des scores -->
      <h4>Scores par tour</h4>
      <table class="score-table">
        <thead>
          <tr>
            <th>Tour</th>
            <th v-for="(turn, index) in maxTurns" :key="'header-' + index">T{{ index + 1 }}</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="player in gameData.players" :key="player">
            <td :style="{ color: getPlayerColor(player) }">{{ player }}</td>
            <td v-for="(score, index) in getPlayedForPlayer(player)" :key="'score-' + index">{{ score }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </template>
  
  
  <script setup>
import { ref, onMounted, computed } from 'vue';
import { Filesystem, Directory } from '@capacitor/filesystem';
import { defineProps } from 'vue';

const props = defineProps({
  fileName: String
});

const gameData = ref(null);
const newMove = ref('');
const maxTurns = ref(0);

const playerColors = ['#4A9FFF', '#F16D6A', '#02BA73', '#DB76E4'];

const currentPlayerColor = computed(() => {
  if (!gameData.value) return '';
  const currentPlayer = gameData.value['current-turn'];
  const playerIndex = gameData.value.players.indexOf(currentPlayer);
  return playerColors[playerIndex] || '#000';
});

const getPlayerColor = (player) => {
  const playerIndex = gameData.value.players.indexOf(player);
  return playerColors[playerIndex] || '#000';
};

const getPlayedForPlayer = (player) => {
  return gameData.value.data[player].played.length > 0 
    ? gameData.value.data[player].played 
    : Array(maxTurns.value).fill('-');
};

const addMove = async () => {
  if (newMove.value === '' || isNaN(newMove.value)) {
    alert('Veuillez entrer un score valide.');
    return;
  }

  const currentPlayer = gameData.value['current-turn'];
  const moveValue = parseInt(newMove.value);

  gameData.value.data[currentPlayer].played.push(moveValue);
  const currentScore = gameData.value.data[currentPlayer].score;
  const newScore = currentScore.length > 0 ? currentScore[currentScore.length - 1] + moveValue : moveValue;
  gameData.value.data[currentPlayer].score.push(newScore);

  const currentPlayerIndex = gameData.value.players.indexOf(currentPlayer);
  const nextPlayerIndex = (currentPlayerIndex + 1) % gameData.value.players.length;
  gameData.value['current-turn'] = gameData.value.players[nextPlayerIndex];

  maxTurns.value = Math.max(
    ...gameData.value.players.map(player => gameData.value.data[player].score.length)
  );

  try {
    await Filesystem.writeFile({
      path: `${props.fileName}.json`,
      data: JSON.stringify(gameData.value, null, 2),
      directory: Directory.Documents,
      encoding: 'utf8'
    });
    newMove.value = '';
    console.log('Mise à jour du fichier JSON réussie');
  } catch (e) {
    console.error('Erreur lors de la mise à jour du fichier JSON :', e);
  }
};

onMounted(async () => {
  try {
    const result = await Filesystem.readFile({
      path: `${props.fileName}.json`,
      directory: Directory.Documents
    });
    gameData.value = JSON.parse(result.data);

    maxTurns.value = Math.max(
      ...gameData.value.players.map(player => gameData.value.data[player].score.length)
    );
  } catch (e) {
    console.error('Erreur lors du chargement du fichier JSON :', e);
  }
});
</script>

  
<style scoped>

  .gameData{
    width: 100%;
  }

  h2{
    color: #004B35;
    font-size: clamp(0px, 25px, 5vw);
    font-weight: 600;
    margin-bottom: 0px;
  }

  h1{
    font-size: clamp(0px, 45px, 9vw);
    font-weight: 600;
    margin-top: 0px;
  }


</style>
  