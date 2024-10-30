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
  top: 30px; /* Décalage correspondant à la hauteur des graduations */
  height: calc(100% - 30px); /* Ajustement de la hauteur pour couvrir les lignes sans empiéter sur les graduations */
  width: 4px;
  background-color: #0665CC;
  z-index: 3; /* Toujours au-dessus des autres éléments */
}

.cursor-head {
  content: "";
  position: absolute;
  top: -10px;
  left: -2px;
  width: 8px;
  height: 10px;
  background-color: #0665CC;
  border-radius: 0 0 4px 4px;
  transform: scaleX(1.4);
}


.cursor-line {
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
}
</style>
