<template>
    <div>
        Moyenne de point par tour
        <div class="average-score-container">
            <span v-for="player in playersData" :key="player.name" class="average-score-item">
                <span class="average-score-playername" :style="{ color: getPlayerColor(player.name) }">
                    {{ player.name }}
                </span>
                <div class="average-score-value" :style="{ backgroundColor: getPlayerColor(player.name) }">
                    {{ player.average.toFixed(2) }}
                </div>
            </span>
        </div>
    </div>
</template>

  
<script setup>
import { computed } from "vue";
import { defineProps } from "vue";

// Props pour récupérer les données
const props = defineProps({
    players: Array, // Liste des joueurs
    playerColors: Array, // Liste des couleurs
    gameData: Object, // Données du jeu
});

// Fonction pour obtenir la couleur d'un joueur
const getPlayerColor = (playerName) => {
    const playerIndex = props.players.indexOf(playerName);
    return props.playerColors[playerIndex] || '#000'; // Noir par défaut si non trouvé
};

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

  

<style scoped>
.average-score-container {
    display: flex;
    flex-wrap: wrap; /* Permet d'adapter les éléments sur plusieurs lignes */
    justify-content: center; /* Centre les éléments horizontalement */
    gap: 20px; /* Espacement entre les joueurs */
    margin-top: 20px;
}

.average-score-item {
    display: flex;
    flex-direction: column; /* Aligne le nom et la valeur verticalement */
    align-items: center; /* Centre les éléments dans chaque colonne */
    text-align: center; /* Centre le texte */
    width: clamp(92.5px, 18.5vw, 120px); /* Largeur responsive */
    height: clamp(92.5px, 18.5vw, 120px); /* Hauteur responsive */
}

.average-score-playername {
    font-weight: bold;
    font-size: clamp(12px, 1.5rem, 2vw); /* Taille responsive */
    margin-bottom: 8px; /* Espacement sous le nom */
}

.average-score-value {
    color: white; /* Texte blanc */
    font-weight: bold;
    font-size: clamp(14px, 1.5rem, 2vw); /* Taille responsive pour la valeur */
    padding: 10px;
    border-radius: 8px; /* Coins arrondis */
    display: flex;
    align-items: center;
    justify-content: center;
}
</style>
