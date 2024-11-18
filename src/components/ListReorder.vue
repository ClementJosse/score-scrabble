<template>
    <div class="player-list-container">
        <div class="number-list">
            <div v-for="(num, index) in numberList" :key="index" class="player-number" :style="{ backgroundColor: playerColors[index] }">
                {{ num }}
            </div>
        </div>
        <draggable v-model="nameList" tag="div" class="textbox-list" :group="{ name: 'players' }" handle=".dragg-element">
            <template #item="{ index }">
                <li class="textbox">
                    <span class="dragg-element">⣿</span>
                    <input
                        v-model="nameList[index]"
                        class="editable-input"
                        :placeholder="`Joueur ${index + 1}`"
                    />
                    <span class="delete-element" @click="handleDelete(index)">x</span>
                </li>
            </template>
        </draggable>
    </div>
    <button @click="addLine" class="add-button" :disabled="nameList.length >= 4">Ajouter une ligne</button>
</template>

<script setup>
import { ref } from 'vue';
import draggable from 'vuedraggable';

const nameList = ref([
    "Joueur 1",
    "Joueur 2",
    "Joueur 3"
]);

// Liste des numéros et couleurs pour chaque joueur
const numberList = ref([1, 2, 3]);
const playerColors = ['#4A9FFF', '#F16D6A', '#02BA73', '#DB76E4'];

const addLine = () => {
    if (nameList.value.length < 4) {
        const newPlayerNumber = numberList.value.length + 1;
        nameList.value.push(`Joueur ${newPlayerNumber}`);
        numberList.value.push(newPlayerNumber);
    }
};

const handleDelete = (index) => {
    if (nameList.value.length > 2) {
        if (nameList.value[index].trim() === '') {
            // Supprimer la textbox et le numéro associé si l'input est vide
            nameList.value.splice(index, 1);
            numberList.value.pop();
        } else {
            // Vider le contenu de l'input si ce n'est pas vide
            nameList.value[index] = '';
        }
    } else {
        // Si seulement deux éléments restent
        if (nameList.value[index].trim() !== '') {
            // Vider le contenu de l'input si ce n'est pas vide, mais ne pas supprimer la textbox
            nameList.value[index] = '';
        } else {
            console.warn("Impossible de supprimer la textbox : il doit rester au moins deux joueurs.");
        }
    }
};
</script>

<style scoped>
.player-list-container {
    display: flex;
    flex-direction: row;
    align-items: flex-start; /* Aligne les éléments au début */
    gap: 10px; /* Espace entre les deux listes */
}

.number-list {
    display: flex;
    flex-direction: column;
    margin-right: 10px; /* Espace à droite des numéros */
}

.player-number {
    display: inline-block;
    width: 30px;
    height: 30px;
    line-height: 30px;
    border-radius: 50%;
    color: white;
    text-align: center;
    font-weight: bold;
    margin-bottom: 5px;
}

.textbox-list {
    display: flex;
    flex-direction: column;
}

.textbox {
    display: flex;
    align-items: center;
    background-color: aqua;
    border-radius: 5px;
    margin-bottom: 5px;
    padding: 5px;
    width: 100%; /* Ajuste la largeur */
    height: 50px; /* Assure que la zone est carrée */
}

.dragg-element {
    margin-right: 10px;
    cursor: grab;
    font-size: 20px;
}

.delete-element {
    margin-left: 10px;
    cursor: pointer;
    color: red;
    font-size: 20px;
}

.editable-input {
    border: none;
    background: transparent;
    width: 100%;
    font-size: inherit;
    outline: none;
}

.add-button {
    margin-top: 10px;
    padding: 8px 16px;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
}

.add-button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
}

.add-button:hover:not(:disabled) {
    background-color: #0056b3;
}
</style>
