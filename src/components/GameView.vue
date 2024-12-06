<template>
  <div>
    <div class="back-container">
      <button @click="$emit('goBack')" class="back-menu">Retour</button>
    </div>
    <div v-if="gameData" class="gameData">
      <div v-if="finisher === gameData.players.length - 1">
        <div>
          <div v-if="gameData?.status === 'ongoing'">
            <button @click="endTheGame" class="end-game-button">Terminer la partie</button>
          </div>
          <div v-else class="turn-info">
            <h2>Gagnant:</h2>
            <h1 v-if="winner" :style="{ color: winner.color }">{{ winner.name.toUpperCase() }}</h1>
            <FinalScore :gameData="gameData" :players="gameData.players" :playerColors="playerColors" />

          </div>
        </div>
      </div>
      <div v-else>
        <div class="turn-info">
          <h2>Au tour de:</h2>
          <h1 :style="{ color: currentPlayerColor }">{{ gameData['current-turn'].toUpperCase() }}</h1>
          <div class="input-move">
            <h4>Score:</h4>
            <div class="input-box">
              <input class="input-text" ref="inputField" v-model="newMove" type="number" placeholder="... "
                @keydown.enter="addMove" :style="{ color: newMove > 0 ? '#004B35' : '#4B0001' }" />
              <button v-if="isValidMove" @click="addMove" class="insert-button">
                <img src="@/assets/enter-value.svg" alt="Ajouter" class="svg-icon" />
              </button>
            </div>
          </div>
        </div>
      </div>


      <!-- Utilisation du composant ScoreTable -->
      <ScoreTable :players="gameData.players" :gameData="gameData" :maxTurns="maxTurns"
        :currentPlayer="gameData['current-turn']" :playerColors="playerColors"
        style="margin-top: clamp(0px,10vw,50px);" />

      <div class="previous-turn-container">
        <button v-if="hasPreviousTurn && gameData?.status === 'ongoing'" @click="previousTurn" class="previous-turn">
          <img src="@/assets/previous-turn.svg" alt="Tour précédent" class="svg-previous-turn" />
          Tour précédent
        </button>
      </div>

      <!-- Graphique des scores -->
      <ScoreGraph :gameData="gameData" :players="gameData.players" :playerColors="playerColors" />

      <AverageScore :gameData="gameData" :players="gameData.players" :playerColors="playerColors" :isFinishing="isFinishing" />
    </div>
  </div>
</template>



<script setup>
import { ref, onMounted, computed, nextTick } from 'vue';
import { Filesystem, Directory, Encoding } from '@capacitor/filesystem';
import ScoreTable from '@/components/ScoreTable.vue';
import ScoreGraph from "@/components/ScoreGraph.vue";
import AverageScore from './AverageScore.vue';
import FinalScore from './FinalScore.vue';
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
const inputField = ref(null);
var bonusPoints = 0;
const finisher = ref(0);

const playerColors = ['#4A9FFF', '#F16D6A', '#02BA73', '#DB76E4'];

const endTheGame = async () => {
  if (!gameData.value) return;

  // Mettre à jour le statut du jeu de manière réactive
  gameData.value = {
    ...gameData.value,
    status: 'ended',
  };

  // Sauvegarder les modifications dans le fichier JSON
  try {
    await Filesystem.writeFile({
      path: `${props.fileName}.json`,
      data: JSON.stringify(gameData.value, null, 2),
      directory: Directory.Documents,
      encoding: Encoding.UTF8
    });
    console.log('Statut de la partie mis à jour : "ended"');
  } catch (e) {
    console.error('Erreur lors de la mise à jour du statut de la partie :', e);
  }
};


const winner = computed(() => {
  if (!gameData.value || !gameData.value.players.length) return null;

  // Trouver le joueur avec le score le plus élevé
  let highestScore = -Infinity;
  let winnerIndex = 0;

  gameData.value.players.forEach((player, index) => {
    const scores = gameData.value.data?.[player]?.score || [];
    const totalScore = scores.length ? scores[scores.length - 1] : 0;

    if (totalScore > highestScore) {
      highestScore = totalScore;
      winnerIndex = index;
    }
  });

  return {
    name: gameData.value.players[winnerIndex],
    color: playerColors[winnerIndex],
  };
});


const isValidMove = computed(() => {
  return newMove.value !== '' && !isNaN(newMove.value);
});

const isFinishing = () => {
  console.log("isFinishing?", finisher.value == gameData.value.players.length - 1)
  return finisher.value == gameData.value.players.length - 1;
};

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
  console.log(gameData.value.data);
  const currentPlayer = gameData.value['current-turn'];
  const currentPlayerIndex = gameData.value.players.indexOf(currentPlayer);

  // Trouver le joueur précédent
  const previousPlayerIndex =
    (currentPlayerIndex - 1 + gameData.value.players.length) %
    gameData.value.players.length;

  const previousPlayer = gameData.value.players[previousPlayerIndex];

  // Vérifier si un bonus a été ajouté
  if (isFinishing()) {
    console.log("Retrait du bonus pour", currentPlayer);
    gameData.value.data[currentPlayer].score.pop()
    const lastPlayed = gameData.value.data[currentPlayer].played.pop()
    bonusPoints -= lastPlayed

    //const enderman = gameData.value.players[(currentPlayerIndex + 1) % gameData.value.players.length];
    //const lastPlayed = gameData.value.data[enderman].played.pop();
    //
    //if (lastPlayed === bonusPoints) {
    //  gameData.value.data[enderman].score.pop(); // Supprime le bonus des scores
    //}
  }

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

    // Recalculer les joueurs avec des scores négatifs
    countPlayersWithNegativeScores();

    // Sauvegarder les modifications dans le fichier JSON
    try {
      await Filesystem.writeFile({
        path: `${props.fileName}.json`,
        data: JSON.stringify(gameData.value, null, 2),
        directory: Directory.Documents,
        encoding: Encoding.UTF8
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

  if (newMove.value < 0) {
    bonusPoints = -calculateNegativeSum();
    console.log(bonusPoints)
    finisher.value = countPlayersWithNegativeScores();
    console.log(finisher)
    if (isFinishing()) {
      console.log("ENDING");
      console.log(gameData.value.players[nextPlayerIndex]);
      var enderman = gameData.value.players[nextPlayerIndex];

      gameData.value.data[enderman].played.push(bonusPoints);
      const currentScore = gameData.value.data[enderman].score;
      const newScore = currentScore.length > 0 ? currentScore[currentScore.length - 1] + bonusPoints : bonusPoints;
      gameData.value.data[enderman].score.push(newScore);
      gameData.value['current-turn'] = enderman;
    }
  }

  try {
    await Filesystem.writeFile({
      path: `${props.fileName}.json`,
      data: JSON.stringify(gameData.value, null, 2),
      directory: Directory.Documents,
      encoding: Encoding.UTF8
    });
    newMove.value = '';
    console.log('Mise à jour du fichier JSON réussie');

    // Garder le focus sur le champ d'entrée
    nextTick(() => {
      if (finisher.value !== gameData.value.players.length - 1) {
        inputField.value.focus(); // Focus après le DOM mis à jour
      }
    });
  } catch (e) {
    console.error('Erreur lors de la mise à jour du fichier JSON :', e);
  }
};


// Fonction principale
const calculateNegativeSum = () => {
  console.log("Début du calcul de la somme des scores négatifs");

  if (!gameData.value || !gameData.value.players) {
    console.error("Données de jeu ou joueurs non définis.");
    return 0;
  }

  const totalNegativeSum = gameData.value.players.reduce((total, player) => {
    const playerNegativeSum = calculatePlayerNegativeSum(player);
    return total + playerNegativeSum;
  }, 0);

  console.log("Somme totale des scores négatifs :", totalNegativeSum);
  return totalNegativeSum;
};

// Fonction pour obtenir les scores d'un joueur
const getPlayerScores = (player) => {
  const played = gameData.value.data?.[player]?.played || [];
  return played;
};

// Fonction pour filtrer les scores négatifs
const filterNegativeScores = (scores) => {
  const negativeScores = scores.filter((score) => typeof score === "number" && score < 0);
  return negativeScores;
};

// Fonction pour calculer la somme des scores négatifs d'un joueur
const calculatePlayerNegativeSum = (player) => {
  const scores = getPlayerScores(player);
  const negativeScores = filterNegativeScores(scores);
  const sum = negativeScores.reduce((acc, score) => acc + score, 0);
  return sum;
};

const countPlayersWithNegativeScores = () => {
  if (!gameData.value || !gameData.value.players) {
    finisher.value = 0; // Réinitialise à 0 si données non définies
    return 0;
  }

  // Compter les joueurs avec au moins un score négatif
  const count = gameData.value.players.filter((player) => {
    const played = gameData.value.data?.[player]?.played || [];
    return played.some((score) => typeof score === "number" && score < 0);
  }).length;

  console.log("Nombre de joueurs avec des scores négatifs :", count);
  finisher.value = count; // Met à jour finisher
  return count;
};


onMounted(async () => {
  try {
    const result = await Filesystem.readFile({
      path: `${props.fileName}.json`,
      directory: Directory.Documents,
      encoding: Encoding.UTF8
    });
    gameData.value = JSON.parse(result.data);

    maxTurns.value = Math.max(
      ...gameData.value.players.map(player => gameData.value.data[player].played.length)
    );

    bonusPoints = -calculateNegativeSum();
    finisher.value = countPlayersWithNegativeScores();

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
  margin-top: 0px;
  padding-top: clamp(0px, 15px, 3vw);
}

.turn-info {
  height: clamp(0px, 208px, 41.6vw);
}

.end-game-button {
  color: #ffffff;
  /* Définit la couleur du texte en vert */
  background-color: #027A56;
  /* Définit la couleur du texte en vert */
  width: 100%;
  height: clamp(0px, 65px, 13vw);
  font-size: clamp(0px, 30px, 6vw);
  /* Ajuste la taille du texte */
  font-weight: 600;
  border-radius: clamp(0px, 10px, 2vw);
  border: 0px;
  margin-bottom: clamp(0px, 80.5px, 16.1vw);
  margin-top: clamp(0px, 80.5px, 16.1vw);
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
  width: fit-content;
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
  font-size: clamp(0px, 40px, 8vw);
  padding-left: clamp(0px, 15px, 3vw);
  width: clamp(0px, 200px, 40vw);
  height: clamp(0px, 80px, 16vw);
}

.input-text {
  border: 1px solid #ddd;
  border-color: #ffffff;
  width: 65%;
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
  width: clamp(0px, 55px, 11vw);
}


.previous-turn-container {
  display: flex;
  justify-content: right;
  height: clamp(0px, 60px, 12vw);
}

.previous-turn {
  display: flex;
  justify-content: right;
  margin-top: clamp(0px, 12px, 2.4vw);
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