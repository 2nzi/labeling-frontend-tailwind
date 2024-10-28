<template>
  <div
    class="timeline-cursor"
    :style="{ left: `${(currentTime - scrollOffset) / visibleDuration * 100}%` }"
  ></div>
</template>

<script>
export default {
  props: ["currentTime", "duration", "visibleDuration", "scrollOffset"],
};
</script>

<style scoped>
.timeline-cursor {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 2px;
  background-color: #1E90FF;
}
</style>
