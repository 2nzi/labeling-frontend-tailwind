<template>
  <div class="timeline-graduation flex justify-between">
    <div
      v-for="mark in graduationMarks"
      :key="mark"
      :style="{ left: `${(mark - scrollOffset) / visibleDuration * 100}%` }"
      class="graduation-mark"
    >
      {{ mark }}
    </div>
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
  },
};
</script>

<style scoped>
.timeline-graduation {
  position: relative;
  height: 20px;
  background-color: #333;
}
.graduation-mark {
  position: absolute;
  color: #ccc;
  font-size: 12px;
}
</style>
