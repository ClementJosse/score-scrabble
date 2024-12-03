<template>
  <div class="game-item" @click="handleView" ref="gameItem">
    <div class="players-list">
      {{ formattedPlayers }}
    </div>
    <div class="game-item-center">
      <div class="creation-date">
        {{ formatFileName(file) }}
      </div>
      <button class="game-actions" @click.stop="handleButtonClick">
        <img v-if="showDeleteButton" src="@/assets/trashcan.svg" alt="delete" class="svg-icon" />
        <img v-else src="@/assets/3dots.svg" alt="more" class="svg-icon" />
      </button>
    </div>
    <div v-if="ongoing" class="current-turn">
      Au tour de:
      {{ props.currentTurn }}
      ({{ props.data[props.currentTurn].score[props.data[props.currentTurn].score.length - 1] || 0 }}pts)
    </div>
    <div v-else class="winner">
      Gagnant:
      {{ winnerName }}
      ({{ props.data[winnerName].score[props.data[winnerName].score.length - 1] || 0 }}pts)
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, computed } from 'vue';
import { defineProps, defineEmits } from 'vue';

// Définir les props que le composant accepte
const props = defineProps({
  ongoing: Boolean, 
  currentTurn: String,
  players: Array,
  data: Object,
  file: String,
});

// Définir les événements que le composant peut émettre
const emit = defineEmits(['view', 'delete']);

// Définir la variable d'état pour le bouton
const showDeleteButton = ref(false);

// Référence au composant pour détecter les clics à l'extérieur
const gameItem = ref(null);

const formatFileName = (fileName) => {
  // Extraire la date et l'heure de la chaîne de caractères
  const [year, month, day, hour, minute] = fileName.replace('.json', '').split('-');

  // Formater la date et l'heure
  const formattedDate = `${day}/${month}/${year.slice(2)}`;
  const formattedTime = `${hour}h${minute}`;

  // Retourner la date et l'heure dans le format désiré
  return `${formattedDate} ${formattedTime}`;
}

const formattedPlayers = computed(() => {
  return props.players.join(' - ');
});

const winnerName = computed(() => {
  if (!props.data || !props.players.length) return null;

  // Trouver le joueur avec le score le plus élevé
  let highestScore = -Infinity;
  let winnerIndex = 0;

  props.players.forEach((player, index) => {
    const scores = props.data[player]?.score || [];
    const totalScore = scores.length ? scores[scores.length - 1] : 0;

    if (totalScore > highestScore) {
      highestScore = totalScore;
      winnerIndex = index;
    }
  });

  return props.players[winnerIndex];
});


// Méthode pour gérer le clic sur le bouton
const handleButtonClick = () => {
  if (showDeleteButton.value) {
    handleDelete(); // Appelle la méthode pour supprimer lorsque le bouton "Supprimer" est visible
  } else {
    showDeleteButton.value = true; // Bascule l'état pour afficher "Supprimer"
  }
};

// Méthode pour gérer les clics en dehors du composant
const handleClickOutside = (event) => {
  if (gameItem.value && !gameItem.value.contains(event.target)) {
    showDeleteButton.value = false;
  }
};

const handleView = () => {
  emit('view', props.file);
};


// Ajouter un écouteur d'événement global pour les clics sur le document
onMounted(() => {
  document.addEventListener('click', handleClickOutside);
});

// Supprimer l'écouteur d'événement lorsqu'on quitte le composant
onBeforeUnmount(() => {
  document.removeEventListener('click', handleClickOutside);
});

// Méthode pour émettre l'événement de suppression
const handleDelete = () => {
  emit('delete', props.file);
};
</script>

<style scoped>
.game-item {
  width: clamp(0px, 80vw, 400px);
  height: clamp(0px, 22vw, 110px);
  display: flex;
  flex-direction: column;
  border: 0px;
  padding: clamp(0px, 15px, 3vw);
  margin-bottom: clamp(0px, 20px, 4vw);
  box-shadow: 0 clamp(0px, 5px, 1vw) clamp(0px, 10px, 2vw) rgba(0, 0, 0, 0.123);
  border-radius: clamp(0px, 10px, 2vw);
}

.players-list {
  display: flex;
  justify-content: center;
  color: #027A56;
  font-size: clamp(0px, 17px, 3.4vw);
  font-weight: 500;
}

.game-item-center {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  color: #004B35;
  font-size: clamp(0px, 30px, 6vw);
  font-weight: 700;
  height: clamp(0px, 45px, 9vw);
}


.game-actions {
  background-color: #ffffffff;
}

.svg-icon {
  width: clamp(0px, 10vw, 50px);
  height: auto;
}

.current-turn, .winner{
  display: flex;
  justify-content: center;
  color: #027A56;
  font-size: clamp(0px, 17px, 3.4vw);
  font-weight: 600;
}
</style>