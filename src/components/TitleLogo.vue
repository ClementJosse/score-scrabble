<template>
  <div class="TitleLogo">
    <img src="@/assets/logo.png" alt="Logo" />
    <span class="title-text">ScoreScrabble</span>
  </div>
</template>

<script>
export default {
  name: 'TitleLogo',
}
</script>

<style scoped>
.TitleLogo {
  display: flex;
  align-items: center;
  justify-content: center;
  max-height: fit-content;
  width: 100%; /* S'assure que la div prend tout l'espace disponible */
}

.TitleLogo img {
  flex: 1; /* Prend 1/4 de l'espace */
  max-width: 25%; /* Limite la largeur à 25% de la div */
  height: auto;
}

.TitleLogo .title-text {
  flex: 3; /* Prend 3/4 de l'espace */
  color: #027A56; /* Définit la couleur du texte en vert */
  font-size: clamp(0px, 40px, 8vw);; /* Ajuste la taille du texte */
  font-weight: 600;
  text-align: left; /* Aligne le texte à gauche */
}
</style>
