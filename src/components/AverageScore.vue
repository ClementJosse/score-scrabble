<template>
    Moyenne de point par tour
    <div class="average-score-container">
      <span
        v-for="player in playersData"
        :key="player.name"
        class="average-score-item"
      >
       {{player.name}} {{ player.average.toFixed(2) }}
      </span>
    </div>
  </template>
  
  <script setup>
  import { computed } from "vue";
  import { defineProps } from "vue";
  
  const props = defineProps({
    players: Array, // Liste des joueurs
    playerColors: Array, // Liste des couleurs
    gameData: Object, // Données du jeu
  });
  



  // Calcul des données des joueurs avec la moyenne filtrée
  const playersData = computed(() =>
    props.players.map((player) => {
      // Filtrer les scores >= 0 et les nombres entiers
      const scores = (props.gameData.data?.[player]?.played || []).filter(
        (score) => score >= 0 && Number.isInteger(score)
      );
      // Calculer la moyenne uniquement avec les scores valides
      const average =
        scores.length > 0
          ? scores.reduce((acc, val) => acc + val, 0) / scores.length
          : 0;
  
      return { name: player, average };
    })
  );
  </script>
  

<style scope>

</style>