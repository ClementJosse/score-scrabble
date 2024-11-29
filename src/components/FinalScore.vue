<template>
    <div>
        <div class="final-score-container">
            <div 
                v-for="player in sortedPlayersData" 
                :key="player.name" 
                class="final-score-item"
            >
                
            <!-- Div pour la valeur du score -->
                <div 
                    class="final-score-value" 
                    :style="{ 
                        backgroundColor: getPlayerColor(player.name), 
                        height: `${calculateHeight(player.final)}px` 
                    }"
                >
                    {{ player.final }}pts
                </div>
                <!-- Div pour le nom du joueur -->
                <div 
                    class="final-score-playername" 
                    :style="{ color: getPlayerColor(player.name) }"
                >
                    {{ player.name }}
                </div>
            </div>
        </div>
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

// Fonction pour obtenir la couleur d'un joueur
const getPlayerColor = (playerName) => {
    const playerIndex = props.players.indexOf(playerName);
    return props.playerColors[playerIndex] || '#000'; // Noir par défaut si non trouvé
};

// Calcul des données des joueurs avec le score final
const playersData = computed(() =>
    props.players.map((player) => {
        const scores = props.gameData.data?.[player]?.score || [];
        const final = scores.length > 0 ? scores[scores.length - 1] : 0;
        return { name: player, final };
    })
);

// Trier les joueurs par score décroissant
const sortedPlayersData = computed(() => {
    console.log([...playersData.value].sort((a, b) => b.final - a.final))
    return [...playersData.value].sort((a, b) => b.final - a.final);
    
});

// Fonction pour calculer la hauteur proportionnelle
const calculateHeight = (score) => {
    const maxScore = sortedPlayersData.value[0]?.final || 1; // Score maximum (évite division par 0)
    const minHeight = 60; // Hauteur minimale (en pixels)
    const maxHeight = 100; // Hauteur maximale (en pixels)
    return Math.max(minHeight, (score / maxScore) * maxHeight); // Proportionne la hauteur
};
</script>


<style scoped>
.final-score-container {
    display: flex;
    justify-content: center; /* Centre les éléments horizontalement */
    gap: clamp(0px, 2vw, 10px); /* Espacement entre les colonnes */
    margin-top: clamp(0px, 2vw, 10px);
}

.final-score-item {
    display: flex;
    flex-direction: column-reverse; /* Aligne le score et le nom verticalement */
    align-items: center; /* Centre les éléments dans chaque colonne */
    text-align: center; /* Centre le texte */
    gap: clamp(0px, 2vw, 10px); /* Espace entre le score et le nom */
}

.final-score-value {
    color: white; /* Texte blanc */
    font-weight: bold;
    font-size: clamp(0px, 4.5vw, 22.5px); /* Taille du texte pour le score */
    border-radius: clamp(0px, 2vw, 10px);
    display: flex;
    justify-content: center;
    width: clamp(0px, 92.5px, 18.5vw);
    padding-top: clamp(0px, 2vw, 10px)
}

.final-score-playername {
    font-weight: bold;
    font-size: clamp(0px, 4vw, 20px); /* Taille du texte pour le nom */
}
</style>
