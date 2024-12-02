<template>
  <div class="game-item">
    <h3>Partie en cours - Tour de : {{ currentTurn }}</h3>
    <div class="players-list">
      <h4>Joueurs :</h4>
      <ul>
        <li v-for="player in players" :key="player">{{ player }}</li>
      </ul>
    </div>
    <div class="scores">
      <h4>Scores :</h4>
      <ul>
        <li v-for="(score, player) in data" :key="player">
          {{ player }} : {{ score.score.reduce((acc, val) => acc + val, 0) }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'GameItem',
  props: {
    currentTurn: {
      type: String,
      required: true
    },
    players: {
      type: Array,
      required: true
    },
    data: {
      type: Object,
      required: true
    }
  }
};
</script>

<style scoped>
.game-item {
  border: 1px solid #ccc;
  padding: 15px;
  margin-bottom: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}
.players-list ul,
.scores ul {
  list-style-type: none;
  padding: 0;
}
</style>