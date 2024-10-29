<template>
  <div class="timeline-event-row" :class="{ 'selected-row': isSelected }" @click="selectEvent">
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

    <!-- Barre de navigation (minimap) ajoutée uniquement à la dernière ligne d'événement -->
    <div v-if="isLastRow" class="timeline-minimap">
      <div
        class="minimap-view"
        :style="{
          width: `${(visibleDuration / duration) * 100}%`,
          left: `${(scrollOffset / duration) * 100}%`
        }"
        @mousedown="startDraggingMinimap"
      >
        <!-- Poignées de la minimap -->
        <div
          class="minimap-handle left-handle"
          @mousedown.stop.prevent="startResizingMinimapLeft"
        ></div>
        <div
          class="minimap-handle right-handle"
          @mousedown.stop.prevent="startResizingMinimapRight"
        ></div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ["event", "isSelected", "selectedBlock", "scrollOffset", "visibleDuration", "duration", "isLastRow"],
  data() {
    return {
      isEditing: false,
      editedName: this.event.name,
      draggingMinimap: false,
      resizingMinimapDirection: null, // 'left' ou 'right' pour savoir quel côté est redimensionné
      startX: 0,
      originalOffset: 0,
      originalDuration: 0,
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
    startDraggingMinimap(event) {
      this.draggingMinimap = true;
      this.startX = event.clientX;
      this.originalOffset = this.scrollOffset;
      document.addEventListener("mousemove", this.dragMinimap);
      document.addEventListener("mouseup", this.stopDraggingMinimap);
    },
    dragMinimap(event) {
      if (this.draggingMinimap) {
        const dx = ((event.clientX - this.startX) / this.$el.clientWidth) * this.duration;
        this.$emit("updateScrollOffset", Math.min(this.duration - this.visibleDuration, Math.max(0, this.originalOffset + dx)));
      }
    },
    stopDraggingMinimap() {
      this.draggingMinimap = false;
      document.removeEventListener("mousemove", this.dragMinimap);
      document.removeEventListener("mouseup", this.stopDraggingMinimap);
    },
    startResizingMinimapLeft(event) {
      this.resizingMinimapDirection = "left";
      this.startX = event.clientX;
      this.originalOffset = this.scrollOffset;
      this.originalDuration = this.visibleDuration;
      document.addEventListener("mousemove", this.resizeMinimap);
      document.addEventListener("mouseup", this.stopResizingMinimap);
    },
    startResizingMinimapRight(event) {
      this.resizingMinimapDirection = "right";
      this.startX = event.clientX;
      this.originalDuration = this.visibleDuration;
      document.addEventListener("mousemove", this.resizeMinimap);
      document.addEventListener("mouseup", this.stopResizingMinimap);
    },
    resizeMinimap(event) {
      const dx = ((event.clientX - this.startX) / this.$el.clientWidth) * this.duration;
      if (this.resizingMinimapDirection === "left") {
        const newOffset = Math.max(0, this.originalOffset + dx);
        const newDuration = Math.max(1, this.originalDuration - dx);
        if (newOffset + newDuration <= this.duration) {
          this.$emit("updateScrollOffset", newOffset);
          this.$emit("updateVisibleDuration", newDuration);
        }
      } else if (this.resizingMinimapDirection === "right") {
        const newDuration = Math.max(1, this.originalDuration + dx);
        if (this.scrollOffset + newDuration <= this.duration) {
          this.$emit("updateVisibleDuration", newDuration);
        }
      }
    },
    stopResizingMinimap() {
      this.resizingMinimapDirection = null;
      document.removeEventListener("mousemove", this.resizeMinimap);
      document.removeEventListener("mouseup", this.stopResizingMinimap);
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

/* Minimap de navigation */
.timeline-minimap {
  position: absolute;
  bottom: -10px;
  width: 100%;
  height: 8px;
  background-color: #333;
  margin-top: 5px;
  margin-left: -10px;
}

.minimap-view {
  position: absolute;
  height: 100%;
  background-color: #989898;
  opacity: 0.7;
  border-radius: 4px;
  display: flex;
  align-items: center;
}

/* Poignées de la minimap */
.minimap-handle {
  width: 8px;
  height: 100%;
  background-color: #ccc;
  border-radius: 50%;
  border: 2px solid #555;
}

.left-handle {
  position: absolute;
  left: 0;
}

.right-handle {
  position: absolute;
  right: 0;
}
</style>
