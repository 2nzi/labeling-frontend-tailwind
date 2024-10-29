<template>
  <div
    class="timeline-event-row"
      :class="{ 'selected-row': isSelected }"
      @click="selectEvent"
    >

    <div class="timeline-minimap">
      <div
        class="minimap-view"
        :style="{
          width: `${(visibleDuration / duration) * 100}%`,
          left: `${(scrollOffset / duration) * 100}%`
        }"
      ></div>
    </div>

    <!-- Nom de l'événement -->
    <span v-if="!isEditing" class="event-name" @dblclick="editEventName">{{ event.name }}</span>
    <input
      v-else
      type="text"
      v-model="editedName"
      @blur="saveEventName"
      @keyup.enter="saveEventName"
      @keyup.esc="cancelEdit"
      class="event-name-input"
    />

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

      <!-- Poignée de redimensionnement gauche -->
      <div 
        class="resize-handle left-handle" 
        @mousedown.stop.prevent="startResizingLeft(index, $event)"
      ></div>

      <!-- Poignée de redimensionnement droite -->
      <div 
        class="resize-handle right-handle" 
        @mousedown.stop.prevent="startResizingRight(index, $event)"
      ></div>
    </div>
  </div>
</template>

<script>
export default {
  props: ["event", "isSelected", "selectedBlock", "scrollOffset", "visibleDuration"],
  data() {
    return {
      isEditing: false,
      editedName: this.event.name,
      draggingBlockIndex: null,
      resizingBlockIndex: null,
      startX: 0,
      originalStart: 0,
      originalDuration: 0,
      resizingDirection: null, // 'left' ou 'right' pour savoir quel côté est redimensionné
    };
  },

  methods: {
    selectEvent() {
      if (!this.isEditing) {
        this.$emit("select");
      }
    },
    editEventName() {
      this.isEditing = true;
      this.editedName = this.event.name;
      this.$emit("editingStatus", true);
      this.$nextTick(() => {
        this.$el.querySelector("input").focus();
      });
    },
    saveEventName() {
      if (this.editedName.trim() !== "") {
        this.$emit("updateEventName", this.editedName);
      }
      this.isEditing = false;
      this.$emit("editingStatus", false);
    },
    cancelEdit() {
      this.isEditing = false;
      this.editedName = this.event.name;
      this.$emit("editingStatus", false);
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
      this.$emit("selectBlock", this.event.blocks[index], this.event);
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

    // Commence le redimensionnement depuis la gauche
    startResizingLeft(index, event) {
      this.resizingBlockIndex = index;
      this.startX = event.clientX;
      this.originalStart = this.event.blocks[index].start;
      this.originalDuration = this.event.blocks[index].duration;
      this.resizingDirection = 'left';
      document.addEventListener("mousemove", this.resizeBlock);
      document.addEventListener("mouseup", this.stopResizingBlock);
    },

    // Commence le redimensionnement depuis la droite
    startResizingRight(index, event) {
      this.resizingBlockIndex = index;
      this.startX = event.clientX;
      this.originalDuration = this.event.blocks[index].duration;
      this.resizingDirection = 'right';
      document.addEventListener("mousemove", this.resizeBlock);
      document.addEventListener("mouseup", this.stopResizingBlock);
    },

    resizeBlock(event) {
      if (this.resizingBlockIndex !== null) {
        const dx = (event.clientX - this.startX) / this.$el.clientWidth;
        const block = this.event.blocks[this.resizingBlockIndex];

        if (this.resizingDirection === 'left') {
          // Redimensionne depuis la gauche : ajuste la position de début et la durée
          const newStart = Math.max(0, this.originalStart + dx * this.visibleDuration);
          const newDuration = Math.max(0.5, this.originalDuration - dx * this.visibleDuration);
          if (newDuration > 0.5) {
            block.start = newStart;
            block.duration = newDuration;
            this.$emit("updateBlockStart", this.resizingBlockIndex, newStart);
            this.$emit("updateBlockDuration", this.resizingBlockIndex, newDuration);
          }
        } else if (this.resizingDirection === 'right') {
          // Redimensionne depuis la droite : ajuste seulement la durée
          const newDuration = Math.max(0.5, this.originalDuration + dx * this.visibleDuration);
          block.duration = newDuration;
          this.$emit("updateBlockDuration", this.resizingBlockIndex, newDuration);
        }
      }
    },

    stopResizingBlock() {
      this.resizingBlockIndex = null;
      this.resizingDirection = null;
      document.removeEventListener("mousemove", this.resizeBlock);
      document.removeEventListener("mouseup", this.stopResizingBlock);
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
.event-name-input {
  width: 100%;
  padding: 4px;
  background-color: #333;
  color: white;
  border: 1px solid #555;
  border-radius: 4px;
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

/* Poignées de redimensionnement */
.resize-handle {
  position: absolute;
  width: 5px;
  height: 100%;
  cursor: ew-resize;
  background-color: #ffffff66;
}

.left-handle {
  left: 0;
  cursor: w-resize;
}

.right-handle {
  right: 0;
  cursor: e-resize;
}


</style>
