<template>
  <div class="timeline-graduation relative w-full flex justify-between">
    <div
      v-for="tick in visibleTicks"
      :key="tick"
      :style="{ left: `${tick.position}%` }"
      class="timeline-tick"
    >
      <span>{{ tick.label }}</span>
    </div>
  </div>
</template>

<script>
export default {
  props: ["duration", "zoomLevel", "scrollOffset"],
  computed: {
    visibleTicks() {
      const ticks = [];
      const visibleDuration = this.duration / this.zoomLevel;
      const interval = this.getTickInterval();
      const startTime = this.scrollOffset;
      const endTime = this.scrollOffset + visibleDuration;

      for (let i = startTime; i <= endTime; i += interval) {
        const relativePosition = ((i - this.scrollOffset) / visibleDuration) * 100;
        ticks.push({
          label: Math.floor(i),
          position: relativePosition,
        });
      }
      return ticks;
    },
  },
  methods: {
    getTickInterval() {
      if (this.zoomLevel >= 4) return 1;
      if (this.zoomLevel >= 2) return 5;
      return 10;
    },
  },
};
</script>

<style scoped>
.timeline-graduation {
  height: 20px;
  background-color: #333;
}
.timeline-tick {
  position: absolute;
  top: 0;
  font-size: 10px;
  color: #ccc;
}
</style>
