<template>
  <div class="editor-container">
    <VideoPlayer
      ref="videoPlayer"
      :currentTime="currentTime"
      :isPlaying="isPlaying"
      @updateCurrentTime="syncTimelineWithVideo"
      @togglePlayPause="togglePlayPause"
      @videoUploaded="handleVideoUploaded"
      @videoDuration="updateDurationFromVideo"
    />

    <!-- Conteneur de la timeline -->
    <div
      class="timeline-editor mx-auto bg-[#464646] text-white p-4 rounded-lg mt-4"
      @wheel="handleWheel"
      @keydown.delete="handleDeleteKey"
      @keydown.a="handleAKey"
      tabindex="0"
    >
      <TimelineControls 
        :isPlaying="isPlaying" 
        :currentTime="currentTime" 
        :isVideoLoaded="isVideoLoaded"
        @togglePlayPause="togglePlayPause" 
        @zoomIn="zoomIn" 
        @zoomOut="zoomOut" 
        @scrollLeft="scrollLeft" 
        @scrollRight="scrollRight"
        @sportSelected="loadSportConfiguration"
      />

      <div class="relative mt-4 w-full overflow-hidden timeline-container" @click.stop="">
        <TimelineGraduation
          :duration="duration"
          :visibleDuration="visibleDuration"
          :scrollOffset="scrollOffset"
          @timeSelected="setCursorTime"
        />

        <TimelineCursor
          :currentTime="currentTime"
          :duration="duration"
          :visibleDuration="visibleDuration"
          :scrollOffset="scrollOffset"
          :rowCount="events.length + 1"
        />

        <div class="event-rows">
          <TimelineEventRow
            v-for="(event, index) in events"
            :key="index"
            :event="event"
            :isSelected="selectedEvent === index"
            :selectedBlock="selectedBlock"
            :scrollOffset="scrollOffset"
            :visibleDuration="visibleDuration"
            :duration="duration"
            :isLastRow="index === events.length - 1"
            @select="selectEvent(index)"
            @selectBlock="selectBlock"
            @updateBlockStart="updateBlockStart"
            @editingStatus="updateEditingStatus"
            @updateEventName="updateEventName"
            @updateBlockDuration="updateBlockDuration"
            @updateScrollOffset="updateScrollOffset"
            @updateVisibleDuration="updateVisibleDuration"
          />

          <!-- Bouton d'ajout de ligne -->
          <div class="add-event-row" @click="addEvent">
            + Ajouter un événement
          </div>
        </div>
      </div>

      <ConfirmationDialog
        :visible="showConfirmation"
        message="Êtes-vous sûr de vouloir supprimer cet événement ?"
        @confirm="deleteSelectedEvent"
        @cancel="showConfirmation = false"
      />
    </div>
  </div>
</template>

<script>
// Import des composants nécessaires
import TimelineControls from './TimelineControls.vue';
import TimelineGraduation from './TimelineGraduation.vue';
import TimelineCursor from './TimelineCursor.vue';
import TimelineEventRow from './TimelineEventRow.vue';
import ConfirmationDialog from './ConfirmationDialog.vue';
import VideoPlayer from './VideoPlayer.vue';
import sportsConfigurations from "@/assets/sportsConfigurations.js";

export default {
  components: {
    TimelineControls,
    TimelineGraduation,
    TimelineCursor,
    TimelineEventRow,
    ConfirmationDialog,
    VideoPlayer,
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
      isVideoLoaded: false,
      animationFrameId: null,
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
      if (this.isVideoLoaded) {
        this.isPlaying = !this.isPlaying;
        this.$refs.videoPlayer.togglePlayback();
        if (this.isPlaying) {
          this.startAnimation();
        } else {
          this.stopAnimation();
        }
      }
    },
    startAnimation() {
      const animateCursor = () => {
        if (this.$refs.videoPlayer && this.isPlaying) {
          this.currentTime = this.$refs.videoPlayer.$refs.video.currentTime;
          this.animationFrameId = requestAnimationFrame(animateCursor);
        }
      };
      this.animationFrameId = requestAnimationFrame(animateCursor);
    },
    stopAnimation() {
      if (this.animationFrameId) {
        cancelAnimationFrame(this.animationFrameId);
        this.animationFrameId = null;
      }
    },
    updateDurationFromVideo(videoDuration) {
      this.duration = videoDuration;
    },
    syncTimelineWithVideo(time) {
      this.currentTime = time;
    },
    handleVideoUploaded() {
      this.isVideoLoaded = true;
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
    updateScrollOffset(newOffset) {
      this.scrollOffset = newOffset;
    },
    updateVisibleDuration(newDuration) {
      this.visibleDuration = newDuration;
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
    this.loadSportConfiguration("Surf");
  },
  beforeUnmount() {
    this.stopAnimation();
    document.removeEventListener("keydown", this.handleKeyEvents);
    document.removeEventListener("click", this.deselectAll);
  },
};
</script>

<style scoped>
.editor-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.timeline-editor {
  width: 100%;
  max-width: 800px;
}
.timeline-cursor {
  z-index: 4;
}
.event-rows {
  background-color: #202020;
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

.timeline-graduation {
  position: relative;
  height: 30px;
  background-color: #202020;
  border-bottom: 3px solid #555;
}

.graduation-mark {
  position: absolute;
  color: #ccc;
  font-size: 12px;
  top: 50%;
  transform: translateY(-50%);
  display: flex;
  align-items: center;
  justify-content: center;
}


</style>
