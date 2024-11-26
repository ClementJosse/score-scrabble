<template>
  <h4>Nom des joueurs :</h4>
  <div class="player-list-button-container">
    <div class="player-list-container">
      <div class="number-list">
        <div v-for="(num, index) in numberList" :key="index" class="player-number"
          :style="{ backgroundColor: playerColors[index] }">
          {{ num }}
        </div>
      </div>
      <draggable v-model="nameList" tag="div" class="textbox-list" :group="{ name: 'players' }" handle=".dragg-element">
        <template #item="{ index }">
          <li class="textbox">
            <span class="dragg-element">⠿</span>
            <div class="input-box">
              <input v-model="nameList[index]" class="editable-input" :placeholder="`Joueur ${index + 1}`" />
              <span class="delete-element" @click="handleDelete(index)">✕</span>
            </div>
          </li>
        </template>
      </draggable>
    </div>
    <div class="button-container">
      <button @click="addLine" class="add-button" :disabled="nameList.length >= 4">+</button>
    </div>
  </div>
  <p>La partie se déroulera selon l’ordre des joueurs indiqué.</p>
  <button @click="createJsonFile" class="new-game" :disabled="isStartButtonDisabled">Commencer la partie</button>
</template>

<script setup>
import { ref, computed } from 'vue';
import draggable from 'vuedraggable';
import { Filesystem, Directory, Encoding } from '@capacitor/filesystem';
import { defineEmits } from 'vue';

const emit = defineEmits(['jsonCreated']);
const nameList = ref(["", ""]);
const numberList = ref([1, 2]);
const playerColors = ['#4A9FFF', '#F16D6A', '#02BA73', '#DB76E4'];

const addLine = () => {
  if (nameList.value.length < 4) {
    nameList.value.push('');
    numberList.value.push(numberList.value.length + 1);
  }
};

const handleDelete = (index) => {
  if (nameList.value.length > 2) {
    if (nameList.value[index].trim() === '') {
      nameList.value.splice(index, 1);
      numberList.value.pop();
    } else {
      nameList.value[index] = '';
    }
  } else {
    if (nameList.value[index].trim() !== '') {
      nameList.value[index] = '';
    } else {
      console.warn("Impossible de supprimer la textbox : il doit rester au moins deux joueurs.");
    }
  }
};

const isStartButtonDisabled = computed(() => {
  return nameList.value.some(name => name.trim() === '');
});

const getFormattedDate = () => {
  const now = new Date();
  const yyyy = now.getFullYear();
  const mm = String(now.getMonth() + 1).padStart(2, '0');
  const dd = String(now.getDate()).padStart(2, '0');
  const hh = String(now.getHours()).padStart(2, '0');
  const min = String(now.getMinutes()).padStart(2, '0');
  const ss = String(now.getSeconds()).padStart(2, '0');
  const ms = String(now.getMilliseconds()).padStart(3, '0');
  return `${yyyy}-${mm}-${dd}-${hh}-${min}-${ss}-${ms}`;
};

const createJsonFile = async () => {
  const trimmedNameList = nameList.value.map(name => name.trim());
  const data = {
    status: "ongoing",
    players: trimmedNameList,
    "current-turn": trimmedNameList[0] || "Joueur 1",
    data: trimmedNameList.reduce((acc, name) => {
      acc[name] = { played: [], score: [] };
      return acc;
    }, {})
  };

  const fileName = getFormattedDate() + '.json';
  try {
    await Filesystem.writeFile({
      path: fileName,
      data: JSON.stringify(data, null, 2),
      directory: Directory.Documents,
      encoding: Encoding.UTF8
    });
    console.log('Fichier JSON créé avec succès :', fileName);

    // Lire et afficher le contenu du fichier JSON dans la console du navigateur
    const result = await Filesystem.readFile({
      path: fileName,
      directory: Directory.Documents
    });
    console.log('Contenu du fichier JSON :', JSON.parse(result.data));

    emit('jsonCreated', fileName.replace('.json', '')); // Émettre l'événement avec le nom du fichier sans extension
  } catch (e) {
    console.error('Erreur lors de la création ou de la lecture du fichier JSON :', e);
  }
};
</script>

<style scoped>
h4 {
  font-size: clamp(0px, 25px, 5vw);
  color: #000000;
  font-weight: 500;
  margin-bottom: clamp(0px, 25px, 5vw);
  margin-top: clamp(0px, 125px, 25vw);
}

.player-list-button-container {
  height: clamp(0px, 300px, 60vw);
}

.player-list-container {
  display: flex;
  flex-direction: row;
  align-items: flex-start;
  /* Aligne les éléments au début */
  gap: clamp(0px, 15px, 3vw);
  /* Espace entre les deux listes */
}

.number-list {
  display: flex;
  flex-direction: column;
  gap: clamp(0px, 15px, 3vw);
}

.player-number {
  display: flex;
  justify-content: center;
  align-items: center;
  width: clamp(0px, 55px, 11vw);
  height: clamp(0px, 55px, 11vw);
  border-radius: 50%;
  color: white;
  text-align: center;
  font-weight: bold;
  font-size: clamp(0px, 30px, 6vw);
}

.textbox-list {
  display: flex;
  flex-direction: column;
  width: 100%;
  gap: clamp(0px, 15px, 3vw);
}

.textbox {
  display: flex;
  align-items: center;
  border-radius: clamp(0px, 10px, 2vw);
  width: 100%;
  /* Ajuste la largeur */
  height: clamp(0px, 55px, 11vw);
}

.dragg-element {
  margin-right: clamp(0px, 15px, 3vw);
  cursor: grab;
  font-size: clamp(0px, 30px, 6vw);
  color: #004B35;
  user-select: none;
  /* Empêche la sélection du texte */
}

.input-box {
  display: flex;
  width: 100%;
  background-color: #ffffff;
  border-radius: clamp(0px, 10px, 2vw);
  border: 0px solid #ccc;
  box-shadow: 0 clamp(0px, 5px, 1vw) clamp(0px, 10px, 2vw) rgba(0, 0, 0, 0.123);
  height: 100%;
  font-size: clamp(0px, 25px, 5vw);
  padding-left: clamp(0px, 15px, 3vw);
}

.delete-element {
  display: flex;
  align-items: center;
  cursor: pointer;
  color: #ADADAD;
  font-size: clamp(0px, 15px, 3vw);
  font-weight: 800;
  margin-left: clamp(0px, 15px, 3vw);
  margin-right: clamp(0px, 15px, 3vw);
}

.editable-input {
  border: none;
  background: transparent;
  width: 100%;
  font-size: inherit;
  outline: none;
  font-weight: 600;
  color: #004B35;
}

.editable-input::placeholder {
  color: #ADADAD;
  /* Couleur bleue pour le texte du placeholder */
  font-weight: 500;
}

.button-container {
  display: flex;
  justify-content: center;
  /* Centre le bouton horizontalement */
  margin-top: clamp(0px, 15px, 3vw);
  /* Ajoute un espacement au-dessus si nécessaire */
}

.add-button {
  width: clamp(0px, 40px, 8vw);
  /* Assurez-vous que la largeur et la hauteur sont égales */
  height: clamp(0px, 40px, 8vw);
  /* Assurez-vous que la largeur et la hauteur sont égales */
  background-color: #027A56;
  color: white;
  border: none;
  border-radius: 50%;
  /* Rendre le bouton circulaire */
  cursor: pointer;
  font-weight: 600;
  font-size: clamp(0px, 40px, 8vw);
  display: flex;
  justify-content: center;
  /* Centre horizontalement le contenu */
  align-items: center;
  /* Centre verticalement le contenu */
}

.add-button:disabled {
  background-color: #ffffff;
  cursor: default;
}

p {
  color: #5F5F5F;
  font-size: clamp(0px, 15px, 3vw);
  margin-top: clamp(0px, 150px, 30vw);
}

.new-game:disabled {
  background-color: #DADADA;
  /* Couleur grise */
  cursor: not-allowed;
  /* Change le curseur en "non permis" */
  color: #ffffff;
  /* Facultatif : change la couleur du texte */
}
</style>
