<template>
  <div class="timeline-editor mx-auto bg-gray-800 text-white p-4 rounded-lg" @wheel="handleWheel">
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
      <TimelineGraduation :duration="duration" :visibleDuration="visibleDuration" :scrollOffset="scrollOffset" />

      <!-- Curseur de lecture -->
      <TimelineCursor :currentTime="currentTime" :duration="duration" :visibleDuration="visibleDuration" :scrollOffset="scrollOffset" />

      <!-- Lignes d'événements -->
      <div class="event-rows">
        <TimelineEventRow v-for="(event, index) in events" :key="index" :event="event" />
      </div>
    </div>
  </div>
</template>

<script>
import TimelineControls from './TimelineControls.vue';
import TimelineGraduation from './TimelineGraduation.vue';
import TimelineCursor from './TimelineCursor.vue';
import TimelineEventRow from './TimelineEventRow.vue';

export default {
  components: {
    TimelineControls,
    TimelineGraduation,
    TimelineCursor,
    TimelineEventRow,
  },
  data() {
    return {
      isPlaying: false,
      currentTime: 0,
      duration: 100,
      visibleDuration: 20,
      scrollOffset: 0,
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
        if (this.currentTime >= this.duration) {
          this.currentTime = 0;
        } else {
          this.currentTime += 0.01;
        }
      }, 10);
    },
    zoomIn() {
      this.updateZoom(this.visibleDuration * 0.95); // Sensibilité plus faible pour zoomer
    },
    zoomOut() {
      this.updateZoom(this.visibleDuration * 1.05); // Sensibilité plus faible pour dézoomer
    },
    updateZoom(newVisibleDuration) {
      const cursorPosition = this.currentTime;

      // Ajuster `scrollOffset` pour centrer le zoom autour du curseur
      this.scrollOffset = cursorPosition - (newVisibleDuration / 2);

      // Empêcher les valeurs négatives et ajuster pour éviter les dépassements
      if (this.scrollOffset < 0) this.scrollOffset = 0;
      if (this.scrollOffset + newVisibleDuration > this.duration) {
        this.scrollOffset = Math.max(0, this.duration - newVisibleDuration);
      }

      this.visibleDuration = newVisibleDuration;
    },
    handleWheel(event) {
      event.preventDefault();
      if (event.deltaY < 0) {
        // Zoom avant
        this.zoomIn();
      } else if (event.deltaY > 0) {
        // Zoom arrière
        this.zoomOut();
      }
    },
    scrollLeft() {
      this.scrollOffset = Math.max(0, this.scrollOffset - this.visibleDuration * 0.1);
    },
    scrollRight() {
      this.scrollOffset = Math.min(this.duration - this.visibleDuration, this.scrollOffset + this.visibleDuration * 0.1);
    },
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
</style>
