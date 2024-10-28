<template>
    <div class="timeline-editor mx-auto bg-gray-800 text-white p-4 rounded-lg">
      <!-- Contrôles de la timeline -->
      <TimelineControls :isPlaying="isPlaying" :currentTime="currentTime" @togglePlayPause="togglePlayPause" />
  
      <!-- Conteneur de la timeline -->
      <div class="relative mt-4 w-full">
        <!-- Graduations -->
        <TimelineGraduation :duration="duration" />
  
        <!-- Curseur de lecture -->
        <TimelineCursor :currentTime="currentTime" :duration="duration" />
  
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
  