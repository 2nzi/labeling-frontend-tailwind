<template>
  <div class="controls flex items-center space-x-4">
    <button @click="$emit('togglePlayPause')">
      <span v-if="!isPlaying">▶</span>
      <span v-else>⏸</span>
    </button>
    <span>{{ formattedTime }}</span>
    <button @click="$emit('zoomIn')">+</button>
    <button @click="$emit('zoomOut')">-</button>
    <button @click="$emit('scrollLeft')">←</button>
    <button @click="$emit('scrollRight')">→</button>
  </div>
</template>

<script>
export default {
  props: ["isPlaying", "currentTime"],
  computed: {
    formattedTime() {
      const minutes = Math.floor(this.currentTime / 60);
      const seconds = Math.floor(this.currentTime % 60);
      const centiseconds = Math.floor((this.currentTime % 1) * 100);
      return `${minutes}:${seconds < 10 ? "0" : ""}${seconds}:${centiseconds < 10 ? "0" : ""}${centiseconds}`;
    },
  },
};
</script>

<style scoped>
.controls button {
  background-color: #555;
  color: white;
  padding: 4px 8px;
  border: none;
  border-radius: 4px;
}
</style>
