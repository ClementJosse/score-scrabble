<template>
  <div class="game-item">
    <div class="players-list">
      {{ props.players }}
    </div>
    <div class="game-item-center">
      <div class="creation-date">
        Date
      </div>
      <div class="suppression-button">
        ...
      </div>
    </div>
    <div v-if="ongoing" class="current-turn">
      Tour actuel: {{ props.currentTurn }}
    </div>
    <div v-else class="winner">
      Gagnant:
    </div>
  </div>
</template>

<script setup>
import { defineProps} from "vue";

const props = defineProps({
  ongoing: Boolean,
  currentTurn: String,
  players: Array,
  data: Object,
});
</script>

<style scoped>
.game-item {
  border: 0px;
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