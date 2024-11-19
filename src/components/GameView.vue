<template>
    <div>
      <button @click="$emit('goBack')" class="back-menu">Retour</button>
      <div v-if="gameData">
        <h2>Au tour de:</h2>
        <h1 :style="{ color: currentPlayerColor }">{{ gameData['current-turn'] }}</h1>
  
        <!-- Input pour ajouter un coup joué -->
        <div class="input-move">
          <input v-model="newMove" type="number" placeholder="Ajouter un coup" />
          <button @click="addMove">Ajouter</button>
        </div>
  
        <div v-for="player in gameData.players" :key="player">
          <h3>{{ player }}</h3>
          <p><strong>Scores :</strong> {{ gameData.data[player].score.join(', ') }}</p>
          <p><strong>Coups joués :</strong> {{ gameData.data[player].played.join(', ') }}</p>
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
  
  const playerColors = ['#4A9FFF', '#F16D6A', '#02BA73', '#DB76E4'];
  
  const currentPlayerColor = computed(() => {
    if (!gameData.value) return '';
    const currentPlayer = gameData.value['current-turn'];
    const playerIndex = gameData.value.players.indexOf(currentPlayer);
    return playerColors[playerIndex] || '#000'; // Défaut à noir si l'index est hors limites
  });
  
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
    } catch (e) {
      console.error('Erreur lors du chargement du fichier JSON :', e);
    }
  });
  </script>
  
  <style scoped>
  .back-menu {
    margin-bottom: 20px;
  }
  </style>
  