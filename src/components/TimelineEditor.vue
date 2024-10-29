<template>
  <div
    class="timeline-editor mx-auto bg-gray-800 text-white p-4 rounded-lg"
    @wheel="handleWheel"
    @keydown.delete="handleDeleteKey"
    @keydown.a="handleAKey"
    tabindex="0"
  >
    <!-- Contrôles de la timeline -->
    <TimelineControls 
      :isPlaying="isPlaying" 
      :currentTime="currentTime" 
      @togglePlayPause="togglePlayPause" 
      @zoomIn="zoomIn" 
      @zoomOut="zoomOut" 
      @scrollLeft="scrollLeft" 
      @scrollRight="scrollRight"
      @sportSelected="loadSportConfiguration"
    />

    
    <!-- Conteneur de la timeline -->
    <div class="relative mt-4 w-full overflow-hidden timeline-container" @click.stop="">
      <!-- Graduations -->
      <TimelineGraduation
        :duration="duration"
        :visibleDuration="visibleDuration"
        :scrollOffset="scrollOffset"
        @timeSelected="setCursorTime"
      />

      <!-- Curseur de lecture avec tête -->
      <TimelineCursor
        :currentTime="currentTime"
        :duration="duration"
        :visibleDuration="visibleDuration"
        :scrollOffset="scrollOffset"
        :rowCount="events.length + 1"
      />

      <!-- Lignes d'événements -->
      <div class="event-rows">
        <TimelineEventRow
          v-for="(event, index) in events"
          :key="index"
          :event="event"
          :isSelected="selectedEvent === index"
          :selectedBlock="selectedBlock"
          :scrollOffset="scrollOffset"
          :visibleDuration="visibleDuration"
          @select="selectEvent(index)"
          @selectBlock="selectBlock"
          @updateBlockStart="updateBlockStart"
          @editingStatus="updateEditingStatus"
          @updateEventName="updateEventName"
          @updateBlockDuration="updateBlockDuration"
        />
        <!-- Bouton d'ajout de ligne -->
        <div class="add-event-row" @click="addEvent">
          + Ajouter un événement
        </div>
      </div>
    </div>

    <!-- Boîte de dialogue de confirmation pour la suppression d'un événement -->
    <ConfirmationDialog
      :visible="showConfirmation"
      message="Êtes-vous sûr de vouloir supprimer cet événement ?"
      @confirm="deleteSelectedEvent"
      @cancel="showConfirmation = false"
    />
  </div>
</template>

<script>
import TimelineControls from './TimelineControls.vue';
import TimelineGraduation from './TimelineGraduation.vue';
import TimelineCursor from './TimelineCursor.vue';
import TimelineEventRow from './TimelineEventRow.vue';
import ConfirmationDialog from './ConfirmationDialog.vue';
import sportsConfigurations from "@/assets/sportsConfigurations.js";

export default {
  components: {
    TimelineControls,
    TimelineGraduation,
    TimelineCursor,
    TimelineEventRow,
    ConfirmationDialog,
  },
  data() {
    return {
      isPlaying: false,
      currentTime: 0,
      duration: 100,
      visibleDuration: 20,
      scrollOffset: 0,
      selectedEvent: null,
      selectedBlock: null,
      showConfirmation: false,
      isEditingEventName: false,
      events: [],
    };
  },
  methods: {
    loadSportConfiguration(sport) {
      if (sportsConfigurations[sport]) {
        this.events = sportsConfigurations[sport].events.map(eventName => ({
          name: eventName,
          blocks: [],
        }));
      }
    },
    togglePlayPause() {
      this.isPlaying = !this.isPlaying;
      if (this.isPlaying) {
        this.startPlayback();
      } else {
        clearInterval(this.playbackInterval);
      }
    },
    startPlayback() {
      this.playbackInterval = setInterval(() => {
        if (this.currentTime >= this.duration) {
          this.currentTime = 0;
        } else {
          this.currentTime += 0.01;
        }
        const cursorPosition = this.currentTime;
        const endVisibleTime = this.scrollOffset + this.visibleDuration;

        if (cursorPosition > endVisibleTime) {
          this.scrollOffset = cursorPosition - this.visibleDuration + 1;
        }
      }, 10);
    },

    handleKeyEvents(event) {
      if (this.isEditingEventName) return;

      if (event.key === "a") {
        event.preventDefault();
        this.addBlockToSelectedEvent();
      } else if (event.key === " ") {
        event.preventDefault();
        this.togglePlayPause();
      } else if (event.key === "Delete") {
        this.deleteSelectedElement();
      }
    },

    deleteSelectedElement() {
      if (this.selectedBlock) {
        if (this.selectedEvent !== null && this.selectedEvent < this.events.length) {
          const event = this.events[this.selectedEvent];
          const blockIndex = event.blocks.indexOf(this.selectedBlock);
          if (blockIndex !== -1) {
            event.blocks.splice(blockIndex, 1);
            this.selectedBlock = null;
          }
        }
      } else {
        this.showConfirmation = true;
      }
    },
    deleteSelectedEvent() {
      if (this.selectedEvent !== null) {
        this.events.splice(this.selectedEvent, 1);
        this.selectedEvent = null;
        this.showConfirmation = false;
      }
    },
    addBlockToSelectedEvent() {
      if (this.selectedEvent !== null) {
        const event = this.events[this.selectedEvent];
        const blockIndex = event.blocks.length + 1;
        event.blocks.push({
          name: `${event.name}_${blockIndex}`,
          start: this.currentTime,
          duration: 3,
        });
      }
    },
    updateBlockStart(index, newStart) {
      if (this.selectedEvent !== null) {
        this.events[this.selectedEvent].blocks[index].start = newStart;
      }
    },
    updateEventName(newName) {
      if (this.selectedEvent !== null) {
        this.events[this.selectedEvent].name = newName;
      }
    },
    updateEditingStatus(isEditing) {
      this.isEditingEventName = isEditing;
    },
    zoomIn() {
      this.updateZoom(this.visibleDuration * 0.95);
    },
    zoomOut() {
      this.updateZoom(this.visibleDuration * 1.05);
    },
    updateZoom(newVisibleDuration) {
      if (newVisibleDuration > this.duration) {
        newVisibleDuration = this.duration;
      }
      const cursorPosition = this.currentTime;
      this.scrollOffset = cursorPosition - (newVisibleDuration / 2);
      if (this.scrollOffset < 0) this.scrollOffset = 0;
      if (this.scrollOffset + newVisibleDuration > this.duration) {
        this.scrollOffset = Math.max(0, this.duration - newVisibleDuration);
      }
      this.visibleDuration = newVisibleDuration;
    },
    handleWheel(event) {
      event.preventDefault();
      if (event.ctrlKey) {
        if (event.deltaY < 0) {
          this.zoomIn();
        } else if (event.deltaY > 0) {
          this.zoomOut();
        }
      } else {
        this.scrollOffset = Math.max(
          0,
          Math.min(this.scrollOffset + event.deltaY * 0.1, this.duration - this.visibleDuration)
        );
      }
    },
    scrollLeft() {
      this.scrollOffset = Math.max(0, this.scrollOffset - this.visibleDuration * 0.1);
    },
    scrollRight() {
      this.scrollOffset = Math.min(this.duration - this.visibleDuration, this.scrollOffset + this.visibleDuration * 0.1);
    },
    setCursorTime(time) {
      this.currentTime = Math.min(this.duration, Math.max(0, time));
    },
    deselectAll(event) {
      // Vérifie si le clic a eu lieu en dehors de .timeline-editor
      if (!event.target.closest('.timeline-editor')) {
        this.selectedEvent = null;
        this.selectedBlock = null;
      }
    },
    selectEvent(index) {
      this.selectedEvent = index;
      this.selectedBlock = null;
    },
    selectBlock(block, event) {
      const eventIndex = this.events.indexOf(event);
      if (eventIndex !== -1 && eventIndex !== this.selectedEvent) {
        this.selectedEvent = eventIndex;
      }
      this.selectedBlock = block;
    },
    addEvent() {
      const newEventName = `Event${this.events.length + 1}`;
      this.events.push({ name: newEventName, blocks: [] });
    },
    updateBlockDuration(eventIndex, blockIndex, newDuration) {
      if (eventIndex !== null && eventIndex < this.events.length) {
        const event = this.events[eventIndex];
        if (event.blocks && blockIndex < event.blocks.length) {
          event.blocks[blockIndex].duration = newDuration;
        }
      }
    },

  },
  mounted() {
    document.addEventListener("keydown", this.handleKeyEvents);
    document.addEventListener("click", this.deselectAll);
    this.loadSportConfiguration("Surf"); // Charger par défaut la configuration "Surf"
  },
  beforeUnmount() {
    clearInterval(this.playbackInterval);
    document.removeEventListener("keydown", this.handleKeyEvents);
    document.removeEventListener("click", this.deselectAll);
  },
};
</script>

<style scoped>
.timeline-editor {
  width: 65%;
  position: relative;
}
.timeline-cursor {
  z-index: 4;
}
.event-rows {
  background-color: #333;
}
.add-event-row {
  background-color: #444;
  color: #aaa;
  padding: 8px;
  text-align: center;
  cursor: pointer;
  border-top: 1px solid #555;
}
.add-event-row:hover {
  background-color: #555;
  color: #fff;
}
</style>
