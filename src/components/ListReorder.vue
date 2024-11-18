<template>
    <div>
        <draggable v-model="nameList" tag="ol">
            <template #item="{ index }">
                <li class="textbox">
                    <span class="player-number" :style="{ backgroundColor: playerColors[index] }">
                        {{ index + 1 }}
                    </span>
                    <input
                        v-model="nameList[index]"
                        class="editable-input"
                        :placeholder="`Joueur ${index + 1}`"
                    />
                </li>
            </template>
        </draggable>
        <button @click="addLine" class="add-button" :disabled="nameList.length >= 4">Ajouter une ligne</button>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import draggable from 'vuedraggable';

const nameList = ref([
    "Joueur 1",
    "Joueur 2"
]);

// Couleurs pour chaque joueur
const playerColors = ['#4A9FFF', '#F16D6A', '#02BA73', '#DB76E4'];

const addLine = () => {
    if (nameList.value.length < 4) {
        nameList.value.push(`Joueur ${nameList.value.length + 1}`);
    }
};
</script>

<style scoped>
.textbox {
    position: relative;
    background-color: aqua;
    padding: 5px;
    margin-bottom: 5px;
    border-radius: 5px;
    display: flex;
    align-items: center;
}

.player-number {
    display: inline-block;
    width: 30px;
    height: 30px;
    line-height: 30px;
    border-radius: 50%;
    color: white;
    text-align: center;
    margin-right: 10px;
    font-weight: bold;
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
