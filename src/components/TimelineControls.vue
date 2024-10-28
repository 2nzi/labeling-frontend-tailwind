<template>
  <div class="flex items-center space-x-4">
    <button @click="togglePlayPause" class="bg-gray-700 text-white p-2 rounded">
      <span v-if="!isPlaying">▶️</span>
      <span v-else>⏸️</span>
    </button>
    <span class="text-white font-mono">{{ formattedTime }}</span>
  </div>
</template>

<script>
export default {
  name: "TimelineControls",
  props: ["isPlaying", "currentTime"],
  emits: ["togglePlayPause"],
  computed: {
    formattedTime() {
      const minutes = Math.floor(this.currentTime / 60);
      const seconds = Math.floor(this.currentTime % 60);
      const centiseconds = Math.floor((this.currentTime % 1) * 100);
      return `${minutes}:${seconds < 10 ? '0' : ''}${seconds}.${centiseconds < 10 ? '0' : ''}${centiseconds}`;
    },
  },
  methods: {
    togglePlayPause() {
      this.$emit("togglePlayPause");
    },
  },
};
</script>
