<template>
  <div class="timeline-editor mx-auto bg-gray-800 text-white p-4 rounded-lg">
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
      <TimelineGraduation :duration="duration" :zoomLevel="zoomLevel" :scrollOffset="scrollOffset" />

      <!-- Curseur de lecture -->
      <TimelineCursor :currentTime="currentTime" :duration="duration" :zoomLevel="zoomLevel" :scrollOffset="scrollOffset" />

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
      zoomLevel: 1,
      scrollOffset: 0, // Position de défilement de la timeline
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
      if (this.zoomLevel < 5) {
        this.zoomLevel += 0.5;
        this.adjustScrollOffset();
      }
    },
    zoomOut() {
      if (this.zoomLevel > 0.5) {
        this.zoomLevel -= 0.5;
        this.adjustScrollOffset();
      }
    },
    scrollLeft() {
      const scrollStep = 5 / this.zoomLevel;
      this.scrollOffset = Math.max(0, this.scrollOffset - scrollStep);
    },
    scrollRight() {
      const scrollStep = 5 / this.zoomLevel;
      const maxScroll = this.duration - this.getVisibleDuration();
      this.scrollOffset = Math.min(maxScroll, this.scrollOffset + scrollStep);
    },
    getVisibleDuration() {
      return this.duration / this.zoomLevel;
    },
    adjustScrollOffset() {
      const maxScroll = this.duration - this.getVisibleDuration();
      this.scrollOffset = Math.min(this.scrollOffset, maxScroll);
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
</style>
