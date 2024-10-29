<template>
  <div
    class="timeline-cursor"
    :style="{ left: `${cursorLeft}%`, height: `${cursorHeight}px` }"
  >
    <div class="cursor-head"></div>
    <div class="cursor-line"></div>
  </div>
</template>

<script>
export default {
  props: ["currentTime", "duration", "visibleDuration", "scrollOffset", "rowCount"],
  computed: {
    cursorLeft() {
      // Calcul de la position horizontale du curseur en pourcentage
      return ((this.currentTime - this.scrollOffset) / this.visibleDuration) * 100;
    },
    cursorHeight() {
      // Calculer la hauteur totale en fonction du nombre de lignes visibles
      const rowHeight = 50; // Hauteur d'une ligne en pixels
      return (this.rowCount - 1) * rowHeight; // Ne couvre pas la dernière ligne
    }
  }
};
</script>

<style scoped>
.timeline-cursor {
  position: absolute;
  top: 20px; /* Décalage correspondant à la hauteur des graduations */
  height: calc(100% - 20px); /* Ajustement de la hauteur pour couvrir les lignes sans empiéter sur les graduations */
  width: 2px;
  background-color: blue;
  z-index: 3; /* Toujours au-dessus des autres éléments */
}

.cursor-head {
  width: 10px;
  height: 10px;
  background-color: blue;
  border-radius: 50%;
  position: absolute;
  top: -5px;
  left: -4px;
}

.cursor-line {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
}
</style>
