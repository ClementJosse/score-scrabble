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
          <input class="input-text" v-model="newMove" type="number" placeholder="... " @keydown.enter="addMove"
            :style="{ color: newMove > 0 ? '#004B35' : '#4B0001' }" />
          <button v-if="isValidMove" @click="addMove" class="insert-button">
            <img src="@/assets/enter-value.svg" alt="Ajouter" class="svg-icon" />
          </button>
        </div>
      </div>

      <!-- Tableau des scores -->
      <h4>Scores par tour</h4>
      <div class="score-container">
        <div class="tables-wrapper">
          <!-- Tableau des joueurs -->
          <table class="player-table">
            <thead>
              <tr>
                <th class="tour">Tour</th>
              </tr>
            </thead>
            <tbody>
              <td>
                <tr v-for="player in gameData.players" :key="player">
                  <div class="turn-rectangle" :style="{ backgroundColor: isPLayerTurn(player) }"></div>
                  <td class="playername-table" :style="{ color: getPlayerColor(player) }">{{ player }}</td>
                </tr>
              </td>
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
                <tr v-for="player in gameData.players" :key="player">
                  <td v-for="index in maxTurns" :key="'score-' + player + '' + index"
                    :style="{ backgroundColor: index % 2 === 0 ? '#FDFDFD' : '#F8F8F8', width: '20px' }">
                    <span
                      v-if="getPlayedForPlayer(player)[index - 1] !== undefined && getPlayedForPlayer(player)[index - 1] !== '-'"
                      :style="{ color: getScoreColor(getPlayedForPlayer(player)[index - 1]) }">
                      {{ getPlayedForPlayer(player)[index - 1] }}
                    </span>
                    <span v-else>...</span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
      <div class="previous-turn-container">
        <button v-if="hasPreviousTurn" @click="previousTurn" class="previous-turn">
          <img src="@/assets/previous-turn.svg" alt="Tour précédent" class="svg-previous-turn" />
          Tour précédent
        </button>
      </div>
    </div>
  </div>
</template>



<script setup>
import { ref, onMounted, computed, nextTick } from 'vue';
import { Filesystem, Directory } from '@capacitor/filesystem';
import { defineProps } from 'vue';

const props = defineProps({
  fileName: String
});

const gameData = ref(null);
const newMove = ref('');
const maxTurns = ref(6);
const scoreContainer = ref(null);

const playerColors = ['#4A9FFF', '#F16D6A', '#02BA73', '#DB76E4'];

const isValidMove = computed(() => {
  return newMove.value !== '' && !isNaN(newMove.value);
});

const hasPreviousTurn = computed(() => {
  // Vérifie si au moins un joueur a des mouvements enregistrés
  return gameData.value.players.some(
    (player) =>
      gameData.value.data[player].played.length > 0 &&
      gameData.value.data[player].score.length > 0
  );
});

const getScoreColor = (value) => {
  if (value > 0) {
    return 'rgb(0, 75, 53)'; // Vert
  } else if (value <= 0) {
    return 'rgb(75, 0, 1)'; // Rouge
  } else {
    return 'rgb(253, 253, 253)'; // Couleur par défaut
  }
};


// Ajoutez cette fonction pour faire défiler vers la droite
const scrollToRight = () => {
  if (scoreContainer.value) {
    scoreContainer.value.scrollLeft = scoreContainer.value.scrollWidth;
  }
};

const currentPlayerColor = computed(() => {
  if (!gameData.value) return '';
  const currentPlayer = gameData.value['current-turn'];
  const playerIndex = gameData.value.players.indexOf(currentPlayer);
  return playerColors[playerIndex] || '#000';
});

const isPLayerTurn = (player) => {
  return currentPlayerColor.value == getPlayerColor(player)
    ? getPlayerColor(player)
    : "#ffffff"
};

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

const previousTurn = async () => {
  const currentPlayer = gameData.value['current-turn'];
  const currentPlayerIndex = gameData.value.players.indexOf(currentPlayer);

  // Trouver le joueur précédent
  const previousPlayerIndex =
    (currentPlayerIndex - 1 + gameData.value.players.length) %
    gameData.value.players.length;

  const previousPlayer = gameData.value.players[previousPlayerIndex];

  // Vérifier si le joueur précédent a des coups enregistrés
  if (
    gameData.value.data[previousPlayer].played.length > 0 &&
    gameData.value.data[previousPlayer].score.length > 0
  ) {
    // Récupérer le dernier coup et le dernier score
    const lastMove = gameData.value.data[previousPlayer].played.pop();
    gameData.value.data[previousPlayer].score.pop();

    // Mettre le dernier coup dans le champ input-text
    newMove.value = lastMove;

    // Mettre à jour le joueur en cours
    gameData.value['current-turn'] = previousPlayer;

    // Sauvegarder les modifications dans le fichier JSON
    try {
      await Filesystem.writeFile({
        path: `${props.fileName}.json`,
        data: JSON.stringify(gameData.value, null, 2),
        directory: Directory.Documents,
        encoding: 'utf8',
      });
      console.log('Tour précédent annulé avec succès.');
    } catch (e) {
      console.error('Erreur lors de la sauvegarde des données :', e);
    }
  } else {
    console.log("Aucun tour précédent à annuler pour ce joueur.");
  }
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

    // Utilisez nextTick pour vous assurer que l'interface est mise à jour avant de faire défiler
    await nextTick();
    scrollToRight();
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

    // Faites défiler vers la droite lors du chargement des données initiales
    nextTick(() => {
      scrollToRight();
    });
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
  display: flex;
  align-items: flex-start;
}

.player-table {
  border-collapse: collapse;
  min-width: clamp(0px, 100px, 20vw);
  border: 0px;
  margin-right: clamp(0px, 5px, 1vw);
  overflow: hidden;
}

.player-table th,
.player-table td {
  text-align: right;
  white-space: nowrap;
  /* Empêche le texte de se couper */
  background-color: #ffffff;
  border: 0px;
}

.playername-table {
  font-size: clamp(0px, 20px, 4vw);
  font-weight: 600;
  text-overflow: ellipsis;
  overflow: hidden;
  max-width: clamp(0px, 100px, 20vw);
  min-width: clamp(0px, 100px, 20vw);
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

.previous-turn-container {
  display: flex;
  justify-content: right;
  height: clamp(0px, 60px, 12vw);
}

.previous-turn {
  display: flex;
  justify-content: right;
  margin-top: clamp(0px, 12.5px, 2.5vw);
  color: #027A56;
  font-size: clamp(0px, 16.5px, 3.3vw);
  align-items: center;
  background-color: #00000000;
  gap: clamp(0px, 5px, 1vw);
  font-weight: 600;
  background-color: #ffffff;
  width: clamp(0px, 170px, 34vw);
  padding: clamp(0px, 10px, 2vw);
  border-radius: clamp(0px, 10px, 2vw);
}

.previous-turn:active {
  background-color: #DADADA;
  /* Couleur lorsqu'on appuie */
}

.svg-previous-turn {
  width: auto;
  height: clamp(0px, 25px, 5vw);
}
</style>