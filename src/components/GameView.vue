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

      <!-- Utilisation du composant ScoreTable -->
      <ScoreTable :players="gameData.players" :gameData="gameData" :maxTurns="maxTurns"
        :currentPlayer="gameData['current-turn']" :playerColors="playerColors"
        style="margin-top: clamp(0px,10vw,50px);" />

      <div class="previous-turn-container">
        <button v-if="hasPreviousTurn" @click="previousTurn" class="previous-turn">
          <img src="@/assets/previous-turn.svg" alt="Tour précédent" class="svg-previous-turn" />
          Tour précédent
        </button>
      </div>

      <!-- Graphique des scores -->
      <ScoreGraph :gameData="gameData" :players="gameData.players"
        :playerColors="['#4A9FFF', '#F16D6A', '#02BA73', '#DB76E4']" />

      <AverageScore :gameData="gameData" :players="gameData.players"
        :playerColors="['#4A9FFF', '#F16D6A', '#02BA73', '#DB76E4']"/>
    </div>
  </div>
</template>



<script setup>
import { ref, onMounted, computed, nextTick } from 'vue';
import { Filesystem, Directory } from '@capacitor/filesystem';
import ScoreTable from '@/components/ScoreTable.vue';
import ScoreGraph from "@/components/ScoreGraph.vue";
import AverageScore from './AverageScore.vue';
import { defineProps } from 'vue';

const props = defineProps({
  fileName: String,
  players: Array,
  playerColors: Array,
  gameData: Object,
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

const previousTurn = async () => {
  console.log(gameData.value.data)
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
    
    const parsedData = JSON.parse(result.data);
    if (parsedData && parsedData.players && parsedData.data) {
      gameData.value = parsedData;
    } else {
      console.error("Données de jeu invalides :", parsedData);
    }
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