<template>
  <div class="main-container">
    <div v-if="currentView === 'main-menu'" class="main-menu">
      <TitleLogo />
      <button @click="goToPlayerMenu" class="new-game">Nouvelle Partie</button>
      <div class="json-list mt-5">
        <div v-if="ongoingFiles.length > 0">
          <h3>Parties en cours :</h3>
          <div v-for="file in ongoingFiles" :key="file">
            <GameItem
              :file="file"
              :ongoing="true"
              :currentTurn="fileData[file].currentTurn"
              :players="fileData[file].players"
              :data="fileData[file].data"
              @view="viewGame"
              @delete="deleteJsonFile"
            />
          </div>
        </div>
        <div v-if="endedFiles.length > 0">
          <h3>Parties terminées :</h3>
          <div v-for="file in endedFiles" :key="file">
            <GameItem
              :file="file"
              :ongoing="false"
              :currentTurn="fileData[file].currentTurn"
              :players="fileData[file].players"
              :data="fileData[file].data"
              @view="viewGame"
              @delete="deleteJsonFile"
            />
          </div>
        </div>
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
import GameItem from './components/GameItem.vue';
import { Filesystem, Directory } from '@capacitor/filesystem';

export default {
  name: 'App',
  components: {
    TitleLogo,
    ListReorder,
    GameView,
    GameItem
  },
  data() {
    return {
      currentView: 'main-menu', // Vue actuelle (main-menu, player-menu, game-view)
      ongoingFiles: [], // Liste des fichiers JSON avec status 'ongoing'
      endedFiles: [], // Liste des fichiers JSON avec status 'ended'
      fileData: {}, // Données des fichiers JSON
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

        const jsonFiles = result.files
          .map(file => (typeof file === 'string' ? file : file.name))
          .filter(fileName => fileName.endsWith('.json'))
          .sort((a, b) => b.localeCompare(a));

        // Réinitialiser les listes
        this.ongoingFiles = [];
        this.endedFiles = [];
        this.fileData = {};

        for (const fileName of jsonFiles) {
          const fileContent = await Filesystem.readFile({
            path: fileName,
            directory: Directory.Documents
          });

          const fileData = JSON.parse(fileContent.data);
          this.fileData[fileName] = {
            currentTurn: fileData["current-turn"],
            players: fileData.players,
            data: fileData.data
          };

          if (fileData.status === 'ongoing') {
            this.ongoingFiles.push(fileName);
          } else if (fileData.status === 'ended') {
            this.endedFiles.push(fileName);
          }
        }
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

    window.addEventListener('jsonUpdated', this.loadJsonFiles);
  },
  beforeUnmount() {
    window.removeEventListener('jsonUpdated', this.loadJsonFiles);
  }
};
</script>


<style>
html,
body {
  height: auto;
  min-height: 100%;
  overflow: auto !important;
  /* Forcer le défilement vertical */
}

.main-container {
  height: auto !important;
  overflow: auto !important;
}

.main-menu {
  display: flex;
  flex-direction: column;
  /* Dispose les éléments verticalement */
  align-items: center;
  /* Centre les éléments horizontalement */
  width: clamp(0px, 400px, 80vw);
  margin: auto;
  /* Centre horizontalement la div */
  margin-top: clamp(0px, 100px, 20vw);
}

.game-view {
  display: flex;
  flex-direction: column;
  /* Dispose les éléments verticalement */
  width: clamp(0px, 400px, 80vw);
  margin: auto;
  /* Centre horizontalement la div */
}

.json-list {
  margin-top: 20px;
  text-align: left;
}

.new-game {
  color: #ffffff;
  /* Définit la couleur du texte en vert */
  background-color: #027A56;
  /* Définit la couleur du texte en vert */
  width: 100%;
  height: clamp(0px, 65px, 13vw);
  font-size: clamp(0px, 30px, 6vw);
  /* Ajuste la taille du texte */
  font-weight: 600;
  border-radius: clamp(0px, 10px, 2vw);
  border: 0px;
  margin-bottom: clamp(0px, 150px, 30vw);
}

.new-game:active {
  background-color: #006345;
  /* Couleur lorsqu'on appuie */
}

.player-menu {
  width: clamp(0px, 400px, 80vw);
  height: 100vh;
  margin: auto;
}

.back-container {
  display: flex;
}

.back-menu {
  margin-top: clamp(0px, 50px, 10vw);
  width: clamp(0px, 100px, 20vw);
  height: clamp(0px, 50px, 10vw);
  border-radius: clamp(0px, 10px, 2vw);
  border: 0px;
  font-size: clamp(0px, 20px, 4vw);
  /* Ajuste la taille du texte */
  font-weight: 600;
  color: #006345;
  background-color: #ffffff;
}

.back-menu:active {
  background-color: #DADADA;
  /* Couleur lorsqu'on appuie */
}

#app {
  font-family: Helvetica, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>