<template>
  <div id="app">
    <TimelineEditor ref="timelineEditor" @videoUploaded="handleVideoUploaded" />

    <div class="export-controls">
      <button 
        @click="exportAsJSON" 
        class="export-button"
        :disabled="!isVideoLoaded"
      >
        Download JSON
      </button>
    </div>
  </div>
</template>

<script>
import TimelineEditor from './components/TimelineEditor.vue';
import { saveAs } from 'file-saver';

export default {
  components: {
    TimelineEditor,
  },
  data() {
    return {
      isVideoLoaded: false,
    };
  },
  methods: {
    exportAsJSON() {
      const events = this.$refs.timelineEditor?.getEventsForExport();
      if (!events) {
        console.error("TimelineEditor is not loaded yet.");
        return;
      }
      
      const exportData = events.map(event => ({
        event: event.event,
        blocks: event.blocks.map(block => ({
          id: block.id || block.name,
          start: block.start,
          end: block.end
        }))
      }));

      const jsonBlob = new Blob([JSON.stringify(exportData, null, 2)], { type: "application/json" });
      saveAs(jsonBlob, "export_data.json");
    },

    handleVideoUploaded() {
      this.isVideoLoaded = true;
    }
  }
};
</script>

<style scoped>
#app {
  text-align: center;
}
.export-controls {
  display: flex;
  justify-content: center;
  gap: 15px;
  margin-top: 15px;
}

.export-button {
  background: linear-gradient(135deg, #4a90e2, #007acc);
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 20px;
  font-size: 14px;
  font-weight: bold;
  cursor: pointer;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.export-button:hover {
  background: linear-gradient(135deg, #007acc, #005ea0);
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
}

.export-button:disabled {
  background: #d3d3d3;
  color: #a0a0a0;
  box-shadow: none;
  cursor: not-allowed;
}
</style>
