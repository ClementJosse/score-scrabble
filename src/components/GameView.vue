<template>
  <div>
    <div class="back-container">
      <button @click="$emit('goBack')" class="back-menu">Retour</button>
    </div>
    <div v-if="gameData" class="gameData">
      <h2>Au tour de:</h2>
      <h1 :style="{ color: currentPlayerColor }">{{ gameData['current-turn'].toUpperCase() }}</h1>
      <div class="input-move">
        <h4>Score:</h4>
        <div class="input-box">
          <input class="input-text"
            v-model="newMove" 
            type="number" 
            placeholder="... " 
            @keydown.enter="addMove"
            :style="{ color: newMove > 0 ? '#004B35' : '#4B0001' }"
          />
          <button v-if="isValidMove" @click="addMove" class="insert-button">
            <img src="@/assets/enter-value.svg" alt="Ajouter" class="svg-icon" />
          </button>
        </div>
      </div>
  
      <!-- Tableau des scores -->
      <h4>Scores par tour</h4>
      <div class="score-container">
        <table class="score-table">
          <thead>
            <tr>
              <th class="sticky-column">Tour</th>
              <th
                v-for="index in maxTurns"
                :key="'header-' + index"
                :style="{ backgroundColor: index % 2 === 0 ? '#FDFDFD' : '#F8F8F8' }"
              >
                <span v-if="isColumnFilled(index)">T{{ index }}</span>
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="player in gameData.players" :key="player">
              <td class="sticky-column" :style="{ color: getPlayerColor(player) }">{{ player }}</td>
              <td
                v-for="index in maxTurns"
                :key="'score-' + player + '-' + index"
                :style="{ backgroundColor: index % 2 === 0 ? '#FDFDFD' : '#F8F8F8' }"
              >
                <span v-if="getPlayedForPlayer(player)[index - 1] !== undefined">
                  {{ getPlayedForPlayer(player)[index - 1] }}
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
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

const isValidMove = computed(() => {
  return newMove.value !== '' && !isNaN(newMove.value);
});

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
    : [];
};

const isColumnFilled = (columnIndex) => {
  if (!gameData.value) return false;

  // Vérifie si au moins un joueur a une donnée pour cette colonne
  return gameData.value.players.some(player => {
    return gameData.value.data[player].played[columnIndex - 1] !== undefined;
  });
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
    ...gameData.value.players.map(player => gameData.value.data[player].played.length)
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
      ...gameData.value.players.map(player => gameData.value.data[player].played.length)
    );
  } catch (e) {
    console.error('Erreur lors du chargement du fichier JSON :', e);
  }
});
</script>



<style scoped>
/* Désactiver les flèches sur les navigateurs Webkit (Chrome, Edge, Safari) */
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

/* Désactiver les flèches sur Firefox */
input[type="number"] {
  -moz-appearance: textfield;
}

.gameData {
  width: 100%;
  display: flex;
  flex-direction: column;
  justify-items: center;
}

h2 {
  color: #004B35;
  font-size: clamp(0px, 25px, 5vw);
  font-weight: 600;
  margin-bottom: 0px;
}

h1 {
  font-size: clamp(0px, 45px, 9vw);
  font-weight: 600;
  margin: 0;
}

h4 {
  font-size: clamp(0px, 15px, 3vw);
  font-weight: 400;
  color: #5F5F5F;
  margin-bottom: 0;
}

.input-move {
  display: flex;
  justify-content: center;
  /* Centre les éléments dans la div */
  align-items: flex-start;
  /* Aligne les éléments à gauche */
  flex-direction: column;
  margin: 0 auto;
  /* Centre la div horizontalement */
  text-align: left;
  /* Aligne le texte à gauche */
}

.input-move input::placeholder {
  color: #ADADAD;
  /* Couleur du placeholder */
}

.input-box {
  display: flex;
  align-items: center;
  background-color: #ffffff;
  border-radius: clamp(0px, 10px, 2vw);
  border: 0px solid #ccc;
  box-shadow: 0 clamp(0px, 5px, 1vw) clamp(0px, 10px, 2vw) rgba(0, 0, 0, 0.123);
  font-size: clamp(0px, 25px, 5vw);
  padding-left: clamp(0px, 15px, 3vw);
  width: clamp(0px, 125px, 25vw);
  height: clamp(0px, 50px, 10vw);
}

.input-text {
  border: 1px solid #ddd;
  border-color: #ffffff;
  width: 80%;
  padding: 8px;
  font-weight: 600;
}
.input-text:focus {
  outline: none;
  /* Supprime le contour */
  border: none;
  /* Assure qu'aucune bordure ne s'affiche */
}

.insert-button {
  background-color: #ffffff;
  margin-right: clamp(0px, 5px, 1vw);
  margin-bottom: clamp(0px, 5px, 1vw);
  margin-top: clamp(0px, 5px, 1vw);
}

.svg-icon {
  height: 100%;
  width: 100%;
}

.score-container {
  overflow-x: auto; /* Ajoute un défilement horizontal */
  white-space: nowrap; /* Empêche les colonnes de se replier */
}

.score-table {
  border-collapse: collapse;
  width: max-content; /* Ajuste la largeur selon le contenu */
}

.score-table th, .score-table td {
  padding: 8px 12px;
  text-align: center;
  border: 1px solid #ddd;
  white-space: nowrap; /* Empêche le texte de se couper */
}

.sticky-column {
  position: sticky;
  left: 0;
  background-color: #fff; /* Fond blanc pour rester visible */
  z-index: 1;
  box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1); /* Optionnel : effet d'ombre */
}

.score-table {
  background-color: #ffffff;
  border-radius: clamp(0px, 10px, 2vw);
  border: 0px solid #ccc;
  box-shadow: 0 clamp(0px, 5px, 1vw) clamp(0px, 10px, 2vw) rgba(0, 0, 0, 0.123);
  width: 100%;
  /* Ajoutez cette ligne */
}

th,
td {
  height: clamp(0px, 40px, 8vw);
}

th {
  color: #5F5F5F;
  font-weight: 500;
}

td {
  font-weight: 600;
}
</style>