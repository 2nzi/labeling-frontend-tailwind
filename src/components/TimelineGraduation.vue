<template>
  <div class="timeline-graduation" @click="handleClick">
    <!-- Marques principales (chiffres) -->
    <div
      v-for="mark in graduationMarks"
      :key="mark"
      :style="{ left: `${(mark - scrollOffset) / visibleDuration * 100}%` }"
      class="graduation-mark"
    >
      {{ mark }}
    </div>
    <!-- Traits principaux sous chaque chiffre -->
    <div
      v-for="mark in graduationMarks"
      :key="'line-' + mark"
      :style="{ left: `${(mark - scrollOffset) / visibleDuration * 100}%` }"
      class="graduation-line main-line"
    ></div>
    <!-- 5 Traits intermédiaires entre les chiffres -->
    <div
      v-for="mark in allIntermediateMarks"
      :key="'intermediate-' + mark"
      :style="{ left: `${(mark - scrollOffset) / visibleDuration * 100}%` }"
      class="graduation-line intermediate-line"
    ></div>
  </div>
</template>

<script>
export default {
  props: ["duration", "visibleDuration", "scrollOffset"],
  computed: {
    graduationMarks() {
      const step = Math.ceil(this.visibleDuration / 10);
      const start = Math.floor(this.scrollOffset / step) * step;
      const marks = [];
      for (let i = start; i <= this.scrollOffset + this.visibleDuration; i += step) {
        marks.push(i);
      }
      return marks;
    },
    allIntermediateMarks() {
      const step = Math.ceil(this.visibleDuration / 10);
      const start = Math.floor(this.scrollOffset / step) * step;
      const marks = [];

      for (let i = start; i <= this.scrollOffset + this.visibleDuration; i += step) {
        for (let j = 1; j <= 5; j++) {
          const intermediateMark = i + (j * step) / 6; // Crée 5 segments intermédiaires
          if (intermediateMark < this.scrollOffset + this.visibleDuration) {
            marks.push(intermediateMark);
          }
        }
      }
      return marks;
    },
  },
  methods: {
    handleClick(event) {
      const timelineWidth = this.$el.clientWidth;
      const clickPosition = event.offsetX;
      const clickTime =
        this.scrollOffset + (clickPosition / timelineWidth) * this.visibleDuration;
      this.$emit("timeSelected", clickTime);
    },
  },
};
</script>

<style scoped>
.timeline-graduation {
  position: relative;
  height: 30px; /* Augmente la hauteur */
  background-color: #202020;
  border-bottom: 3px solid #555;
}

.graduation-mark {
  position: absolute;
  color: #ccc;
  font-size: 12px;
  top: 10%; /* Centre verticalement */
  transform: translateX(-3px);
  display: flex;
  align-items: center;
  justify-content: center;
}

.graduation-line {
  position: absolute;
  bottom: 0;
  width: 1px;
  background-color: #ccc;
}

.main-line {
  height: 10px; /* Hauteur des traits principaux */
}

.intermediate-line {
  height: 5px; /* Hauteur des traits intermédiaires */
  background-color: #777; /* Couleur plus légère pour les traits intermédiaires */
}
</style>
