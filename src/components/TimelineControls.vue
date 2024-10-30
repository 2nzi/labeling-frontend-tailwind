<template>
  <div class="controls flex items-center justify-between space-x-2">
    <!-- Section des contrôles de lecture -->
    <div class="flex items-center space-x-2">
      <button @click="$emit('togglePlayPause')" class="icon-button">
        <PauseIcon v-if="isPlaying" class="icon-size" />
        <PlayIcon v-else class="icon-size" />
      </button>
      <span class="time-display">{{ formatTime(currentTime) }}</span>
    </div>
    
    <!-- Section de sélection du sport -->
    <div>
      <select @change="sportSelected($event)" class="sport-select">
        <option v-for="(config, sport) in sportsConfigurations" :key="sport" :value="sport">
          {{ sport }}
        </option>
      </select>
    </div>
  </div>
</template>

<script>
import sportsConfigurations from "@/assets/sportsConfigurations.js";
import PlayIcon from './PlayIcon.vue';
import PauseIcon from './PauseIcon.vue';

export default {
  components: {
    PlayIcon,
    PauseIcon,
  },
  props: ["isPlaying", "currentTime"],
  data() {
    return {
      sportsConfigurations,
    };
  },
  methods: {
    formatTime(time) {
      const minutes = Math.floor(time / 60);
      const seconds = Math.floor(time % 60);
      const centiseconds = Math.floor((time % 1) * 100);
      return `${minutes}:${seconds < 10 ? `0${seconds}` : seconds}.${centiseconds < 10 ? `0${centiseconds}` : centiseconds}`;
    },
    sportSelected(event) {
      this.$emit("sportSelected", event.target.value);
    },
  },
};
</script>

<style scoped>
.controls {
  padding: 8px;
}

.icon-button {
  border: none;
  padding: 4px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.icon-size {
  width: 20px;
  height: 20px;
}

.control-button {
  background-color:#464646;
  color: white;
  padding: 4px 6px;
  border: 3px solid #555; /* Contour de 2px avec couleur et style */
  border-radius: 4px;
  font-size: 14px;
}


.time-display {
  font-size: 14px;
  color: white;
}

.sport-select {
  background-color: #464646;
  color: white;
  padding: 4px 8px;
  border: 3px solid #555; /* Définir l'épaisseur, le style, et la couleur de la bordure */
  border-radius: 4px;
  font-size: 0.9em;
}

</style>
