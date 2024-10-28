<template>
  <div
    class="timeline-event-row"
    :class="{ 'selected-row': isSelected }"
    @click="selectEvent"
  >
    <!-- Nom de l'événement -->
    <span class="event-name">{{ event.name }}</span>

    <!-- Blocs de l'événement -->
    <div
      v-for="(block, index) in event.blocks"
      :key="index"
      :style="blockStyle(block)"
      class="event-block"
      :class="{ 'selected-block': selectedBlock === block }"
      @click.stop="selectBlock(block)"
      @mousedown.stop.prevent="startDraggingBlock(index, $event)"
    >
      {{ block.name }}
    </div>
  </div>
</template>

<script>
export default {
  props: ["event", "isSelected", "selectedBlock", "scrollOffset", "visibleDuration"],
  data() {
    return {
      draggingBlockIndex: null,
      startX: 0,
      originalStart: 0,
    };
  },
  methods: {
    selectEvent() {
      this.$emit("select");
    },
    blockStyle(block) {
      const startPercent = ((block.start - this.scrollOffset) / this.visibleDuration) * 100;
      const widthPercent = (block.duration / this.visibleDuration) * 100;
      return {
        left: `${startPercent}%`,
        width: `${widthPercent}%`,
      };
    },
    selectBlock(block) {
      this.$emit("selectBlock", block);
    },
    startDraggingBlock(index, event) {
      this.draggingBlockIndex = index;
      this.startX = event.clientX;
      this.originalStart = this.event.blocks[index].start;
      document.addEventListener("mousemove", this.dragBlock);
      document.addEventListener("mouseup", this.stopDraggingBlock);
    },
    dragBlock(event) {
      if (this.draggingBlockIndex !== null) {
        const dx = (event.clientX - this.startX) / this.$el.clientWidth;
        const newStart = Math.max(0, this.originalStart + dx * this.visibleDuration);
        const maxStart = this.$parent.duration - this.event.blocks[this.draggingBlockIndex].duration;
        this.$emit("updateBlockStart", this.draggingBlockIndex, Math.min(maxStart, newStart));
      }
    },
    stopDraggingBlock() {
      this.draggingBlockIndex = null;
      document.removeEventListener("mousemove", this.dragBlock);
      document.removeEventListener("mouseup", this.stopDraggingBlock);
    },
  },
};
</script>

<style scoped>
.timeline-event-row {
  height: 50px;
  border-bottom: 1px solid #555;
  display: flex;
  align-items: center;
  padding-left: 10px;
  position: relative;
  cursor: pointer;
}
.timeline-event-row.selected-row {
  background-color: #5a5a2b;
  border: 2px solid yellow;
}
.event-name {
  color: white;
  font-weight: bold;
}
.event-block {
  position: absolute;
  top: 5px;
  height: 40px;
  background-color: #ff5733;
  color: white;
  text-align: center;
  padding: 5px;
  border-radius: 4px;
  font-size: 0.9em;
  display: flex;
  align-items: center;
  justify-content: center;
}
.event-block.selected-block {
  border: 2px solid blue;
}
</style>
