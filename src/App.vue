<template>
  <div class="main">
    <TitleLogo />
    <button class="new-game">Nouvelle Partie</button>
  </div>
</template>

<script>
import TitleLogo from './components/TitleLogo.vue'; // Assurez-vous que le chemin est correct

export default {
  name: 'App',
  components: {
    TitleLogo
  }
}
</script>

<style>
.main {
  display: flex;
  flex-direction: column; /* Dispose les éléments verticalement */
  align-items: center; /* Centre les éléments horizontalement */
  width: clamp(0px, 400px, 80vw);
  height: 100vh;
  margin: auto; /* Centre horizontalement la div */
  margin-top: clamp(0px, 100px, 20vw);
}

button {
  color: #ffffff; /* Définit la couleur du texte en vert */
  background-color: #027A56; /* Définit la couleur du texte en vert */
  width: 100%;
  height: clamp(0px, 65px, 13vw);
  font-size: clamp(0px, 30px, 6vw); /* Ajuste la taille du texte */
  font-weight: 600;
  border-radius: clamp(0px, 10px, 2vw);
  border: 0px;
  margin-top: clamp(0px, 200px, 40vw);
  margin-bottom: clamp(0px, 200px, 40vw);
}

button:active {
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
