<template>
  <div
    class="timeline-editor mx-auto bg-gray-800 text-white p-4 rounded-lg"
    @wheel="handleWheel"
    @keydown.delete="confirmDeletion"
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
    />

    <!-- Conteneur de la timeline -->
    <div class="relative mt-4 w-full overflow-hidden">
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
          @select="selectEvent(index)"
        />
        <!-- Bouton d'ajout de ligne -->
        <div class="add-event-row" @click="addEvent">
          + Ajouter un événement
        </div>
      </div>
    </div>

    <!-- Boîte de dialogue de confirmation pour la suppression -->
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
      showConfirmation: false,
      events: [
        { name: "Takeoff" },
        { name: "Carve" },
        { name: "360" },
        { name: "Roller" },
      ],
    };
  },
  methods: {
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
        if (this.currentTime >= this.duration - 5) {
          this.currentTime = this.duration - 5;
          this.isPlaying = false;
          clearInterval(this.playbackInterval);
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
    confirmDeletion() {
      if (this.selectedEvent !== null) {
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
      this.currentTime = Math.min(this.duration - 5, Math.max(0, time));
    },
    selectEvent(index) {
      this.selectedEvent = this.selectedEvent === index ? null : index;
    },
    addEvent() {
      const newEventName = `Event${this.events.length + 1}`;
      this.events.push({ name: newEventName });
    }
  },
  beforeUnmount() {
    clearInterval(this.playbackInterval);
  },
};
</script>

<style scoped>
.timeline-editor {
  width: 65%;
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
