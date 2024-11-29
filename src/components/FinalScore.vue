<template>
    <div>
        <div class="final-score-container">
            <span v-for="player in sortedPlayersData" :key="player.name" class="final-score-item">
                <div class="final-score-value" :style="{ backgroundColor: getPlayerColor(player.name) }">
                    {{ player.final }}pts
                </div>
                <span class="final-score-playername" :style="{ color: getPlayerColor(player.name) }">
                    {{ player.name }}
                </span>
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

// Calcul des données des joueurs avec le score final
const playersData = computed(() =>
    props.players.map((player) => {
        // Récupérer les scores du joueur
        const scores = props.gameData.data?.[player]?.score || [];
        // Prendre le dernier score comme score final (ou 0 s'il n'y a pas de score)
        const final = scores.length > 0 ? scores[scores.length - 1] : 0;

        return { name: player, final };
    })
);

// Trier les joueurs par score décroissant (sans modifier playersData)
const sortedPlayersData = computed(() => {
    return [...playersData.value].sort((a, b) => b.final - a.final);
});
</script>


<style scoped>
.final-score-container {
    display: flex;
    justify-content: center; /* Centre les éléments horizontalement */
    margin-top: clamp(0px, 10px, 2vw);
    gap: clamp(0px, 10px, 2vw);
    margin-bottom: clamp(0px, 60px, 12vw);
}

.final-score-item {
    display: flex;
    flex-direction: column; /* Aligne le nom et la valeur verticalement */
    align-items: center; /* Centre les éléments dans chaque colonne */
    text-align: center; /* Centre le texte */
}

.final-score-playername {
    font-weight: bold;
    font-size: clamp(0px, 20px, 4vw); /* Taille responsive */
    margin-top: 8px; /* Espacement au-dessus du nom */
}

.final-score-value {
    color: white; /* Texte blanc */
    font-weight: bold;
    font-size: clamp(0px, 22.5px, 4.5vw); /* Taille responsive pour la valeur */
    border-radius: clamp(0px, 10px, 2vw);
    display: flex;
    align-items: center;
    justify-content: center;
    height: clamp(0px, 50px, 10vw);
    width: clamp(0px, 92.5px, 18.5vw);
}
</style>
