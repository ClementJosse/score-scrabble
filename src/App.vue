<template>
  <div class="main-container">
    <div v-if="currentView === 'main-menu'" class="main-menu">
      <TitleLogo />
      <button @click="goToPlayerMenu" class="new-game">Nouvelle Partie</button>
      <div class="json-list mt-5">
        <h2>Fichiers JSON créés :</h2>
        <ul>
          <li v-for="file in jsonFiles" :key="file" class="flex items-center justify-between">
            {{ file }}
            <button @click="viewGame(file)" class="ml-2 bg-green-500 text-white px-2 py-1 rounded hover:bg-green-700">
              Ouvrir
            </button>
            <button @click="deleteJsonFile(file)" class="ml-2 bg-red-500 text-white px-2 py-1 rounded hover:bg-red-700">
              Supprimer
            </button>
          </li>
        </ul>
      </div>
    </div>

    <div v-else-if="currentView === 'player-menu'" class="player-menu">
      <div class="back-container">
        <button @click="goToMainMenu" class="back-menu">Retour</button>
      </div>
      <ListReorder @jsonCreated="openGameView" />
    </div>

    <div v-else-if="currentView === 'game-view'" class="game-view">
      <GameView :fileName="selectedFileName" @goBack="goToMainMenu" />
    </div>
  </div>
</template>

<script>
import TitleLogo from './components/TitleLogo.vue';
import ListReorder from './components/ListReorder.vue';
import GameView from './components/GameView.vue';
import { Filesystem, Directory } from '@capacitor/filesystem';

export default {
  name: 'App',
  components: {
    TitleLogo,
    ListReorder,
    GameView
  },
  data() {
    return {
      currentView: 'main-menu', // Vue actuelle (main-menu, player-menu, game-view)
      jsonFiles: [], // Liste des fichiers JSON
      selectedFileName: '' // Nom du fichier JSON sélectionné
    };
  },
  methods: {
    async loadJsonFiles() {
      try {
        const result = await Filesystem.readdir({
          path: '',
          directory: Directory.Documents
        });

        // Extraire le nom des fichiers et trier par ordre décroissant
        this.jsonFiles = result.files
          .map(file => typeof file === 'string' ? file : file.name)
          .filter(fileName => fileName.endsWith('.json'))
          .sort((a, b) => b.localeCompare(a));
      } catch (e) {
        console.error('Erreur lors du chargement des fichiers JSON :', e);
      }
    },
    async deleteJsonFile(fileName) {
      try {
        await Filesystem.deleteFile({
          path: fileName,
          directory: Directory.Documents
        });
        console.log('Fichier JSON supprimé :', fileName);
        this.loadJsonFiles(); // Recharge la liste des fichiers après la suppression
      } catch (e) {
        console.error('Erreur lors de la suppression du fichier JSON :', e);
      }
    },
    goToPlayerMenu() {
      this.currentView = 'player-menu';
    },
    goToMainMenu() {
      this.loadJsonFiles();
      this.currentView = 'main-menu';
    },
    viewGame(fileName) {
      this.selectedFileName = fileName.replace('.json', '');
      this.currentView = 'game-view';
    },
    openGameView(fileName) {
      this.selectedFileName = fileName;
      this.currentView = 'game-view';
    }
  },
  mounted() {
    this.loadJsonFiles(); // Charge la liste des fichiers au montage du composant

    // Écoute l'événement personnalisé 'jsonUpdated' pour actualiser la liste
    window.addEventListener('jsonUpdated', this.loadJsonFiles);
  },
  beforeUnmount() {
    // Nettoyer l'écouteur d'événements lorsque le composant est démonté
    window.removeEventListener('jsonUpdated', this.loadJsonFiles);
  }
};
</script>


<style>
body, html {
  overflow-x: hidden; /* Empêche le scroll horizontal global */
}
.main-container{
  display: flex;
  flex-direction: row;
  overflow-x: hidden;
  transition: transform 0.5s ease-in-out;
}

.main-menu{
  display: flex;
  flex-direction: column; /* Dispose les éléments verticalement */
  align-items: center; /* Centre les éléments horizontalement */
  width: clamp(0px, 400px, 80vw);
  margin: auto; /* Centre horizontalement la div */
  margin-top: clamp(0px, 100px, 20vw);
}

.game-view {
  display: flex;
  flex-direction: column; /* Dispose les éléments verticalement */
  width: clamp(0px, 400px, 80vw);
  margin: auto; /* Centre horizontalement la div */
}

.json-list {
  margin-top: 20px;
  text-align: left;
}

.new-game {
  color: #ffffff; /* Définit la couleur du texte en vert */
  background-color: #027A56; /* Définit la couleur du texte en vert */
  width: 100%;
  height: clamp(0px, 65px, 13vw);
  font-size: clamp(0px, 30px, 6vw); /* Ajuste la taille du texte */
  font-weight: 600;
  border-radius: clamp(0px, 10px, 2vw);
  border: 0px;
  margin-bottom: clamp(0px, 150px, 30vw);
}

.new-game:active {
  background-color: #006345; /* Couleur lorsqu'on appuie */
}

.player-menu{
  width: clamp(0px, 400px, 80vw);
  height: 100vh;
  margin: auto;
}

.back-container{
  display: flex;
}

.back-menu{
  margin-top: clamp(0px, 50px, 10vw);
  width: clamp(0px, 100px, 20vw);
  height: clamp(0px, 50px, 10vw);
  border-radius: clamp(0px, 10px, 2vw);
  border: 0px;
  font-size: clamp(0px, 20px, 4vw); /* Ajuste la taille du texte */
  font-weight: 600;
  color: #006345;
  background-color: #ffffff;
}

.back-menu:active {
  background-color: #DADADA; /* Couleur lorsqu'on appuie */
}

#app {
  font-family: Helvetica, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>