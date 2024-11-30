<template>
    <div>
        <h4 class="average-title">Moyenne de point par tour</h4>
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
    isFinishing: Boolean,
});

// Fonction pour obtenir la couleur d'un joueur
const getPlayerColor = (playerName) => {
    const playerIndex = props.players.indexOf(playerName);
    return props.playerColors[playerIndex] || '#000'; // Noir par défaut si non trouvé
};

// Calcul des données des joueurs avec la moyenne filtrée
const playersData = computed(() =>
    props.players.map((player) => {
        // Obtenir la liste des scores du joueur
        let scores = props.gameData.data?.[player]?.played || [];
        console.log(props.isFinishing);

        // Si isFinishing est vrai et que le dernier score n'est pas négatif, l'exclure
        if (props.isFinishing && scores.length > 0 && scores[scores.length - 1] >= 0) {
            scores = scores.slice(0, -1); // Exclure le dernier score
            console.log(player," est découpé")
        }

        // Filtrer les scores >= 0 et les nombres entiers
        scores = scores.filter((score) => score >= 0 && Number.isInteger(score));
        console.log(scores)
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


.average-title {
    font-size: clamp(0px, 15px, 3vw);
    font-weight: 400;
    text-align: center;
    color: #5F5F5F;
    padding-top: clamp(0px, 25px, 5vw)
}



.average-score-container {
    display: flex;
    justify-content: center; /* Centre les éléments horizontalement */
    margin-top: clamp(0px, 10px, 2vw);
    gap: clamp(0px, 10px, 2vw);
    margin-bottom: clamp(0px, 60px, 12vw);
}

.average-score-item {
    display: flex;
    flex-direction: column; /* Aligne le nom et la valeur verticalement */
    align-items: center; /* Centre les éléments dans chaque colonne */
    text-align: center; /* Centre le texte */
}

.average-score-playername {
    font-weight: bold;
    font-size: clamp(0px, 20px, 4vw); /* Taille responsive */
    margin-bottom: 8px; /* Espacement sous le nom */
}

.average-score-value {
    color: white; /* Texte blanc */
    font-weight: bold;
    font-size: clamp(0px, 20px, 4vw); /* Taille responsive pour la valeur */
    border-radius: clamp(0px, 10px, 2vw);
    display: flex;
    align-items: center;
    justify-content: center;
    height: clamp(0px, 50px, 10vw);
    width: clamp(0px, 92.5px, 18.5vw);
}
</style>
