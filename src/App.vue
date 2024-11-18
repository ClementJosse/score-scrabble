<template>
  <div class="main">
    <TitleLogo />
    <button @click="createJsonFile" class="new-game">Nouvelle Partie</button>
    <div class="json-list mt-5">
      <h2>Fichiers JSON créés :</h2>
      <ul>
        <li v-for="file in jsonFiles" :key="file" class="flex items-center justify-between">
          {{ file }}
          <button @click="deleteJsonFile(file)" class="ml-2 bg-red-500 text-white px-2 py-1 rounded hover:bg-red-700">
            Supprimer
          </button>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import TitleLogo from './components/TitleLogo.vue'; // Assurez-vous que le chemin est correct
import { Filesystem, Directory, Encoding } from '@capacitor/filesystem';

export default {
  name: 'App',
  components: {
    TitleLogo
  },
  data() {
    return {
      jsonFiles: [] // Liste des fichiers JSON
    };
  },
  methods: {
    getFormattedDate() {
      const now = new Date();
      const yyyy = now.getFullYear();
      const mm = String(now.getMonth() + 1).padStart(2, '0');
      const dd = String(now.getDate()).padStart(2, '0');
      const hh = String(now.getHours()).padStart(2, '0');
      const min = String(now.getMinutes()).padStart(2, '0');
      const ss = String(now.getSeconds()).padStart(2, '0');
      const ms = String(now.getMilliseconds()).padStart(3, '0');
      return `${yyyy}-${mm}-${dd}-${hh}-${min}-${ss}-${ms}`;
    },
    async createJsonFile() {
      const data = {
        date: new Date().toISOString(),
        game: "Nouvelle partie",
        score: 0
      };
      const fileName = this.getFormattedDate() + '.json';
      try {
        await Filesystem.writeFile({
          path: fileName,
          data: JSON.stringify(data),
          directory: Directory.Documents,
          encoding: Encoding.UTF8
        });
        console.log('Fichier JSON créé avec succès :', fileName);
        this.loadJsonFiles(); // Recharge la liste des fichiers après la création
      } catch (e) {
        console.error('Erreur lors de la création du fichier JSON :', e);
      }
    },
    async loadJsonFiles() {
      try {
        const result = await Filesystem.readdir({
          path: '',
          directory: Directory.Documents
        });

        // Extraire le nom des fichiers et trier par ordre décroissant
        this.jsonFiles = result.files
          .map(file => typeof file === 'string' ? file : file.name) // S'assurer d'extraire le nom
          .filter(fileName => fileName.endsWith('.json')) // Filtrer uniquement les fichiers JSON
          .sort((a, b) => b.localeCompare(a)); // Trier les fichiers du plus récent au plus ancien
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
    }
  },
  mounted() {
    this.loadJsonFiles(); // Charge la liste des fichiers au montage du composant
  }
}
</script>

<style>
.main {
  display: flex;
  flex-direction: column; /* Dispose les éléments verticalement */
  align-items: center; /* Centre les éléments horizontalement */
  width: clamp(0px, 400px, 80vw);
  margin: auto; /* Centre horizontalement la div */
  margin-top: clamp(0px, 100px, 20vw);
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
  margin-top: clamp(0px, 150px, 30vw);
  margin-bottom: clamp(0px, 150px, 30vw);
}

.new-game:active {
  background-color: #006345; /* Couleur lorsqu'on appuie */
}

#app {
  font-family: Helvetica, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
