<template>
  <div id="app">
    <!-- Toggle Button for switching views -->
    <div class="view-toggle">
      <label class="switch">
        <input type="checkbox" v-model="showMosaicView" />
        <span class="slider"></span>
      </label>
      <span>{{ showMosaicView ? "Mosaic View" : "Timeline Editor" }}</span>
    </div>

    <!-- Show TimelineEditor or MosaicView based on toggle -->
    <TimelineEditor
      v-if="!showMosaicView"
      ref="timelineEditor"
      @videoUploaded="handleVideoUploaded"
    />
    <MosaicView
      v-if="showMosaicView"
      :activeLabel="activeLabel"
      @updateLabel="updateActiveLabel"
    />

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
import MosaicView from './components/MosaicView.vue';
import { saveAs } from 'file-saver';

export default {
  components: {
    TimelineEditor,
    MosaicView,
  },
  data() {
    return {
      isVideoLoaded: false,
      showMosaicView: false, // Toggle state
      activeLabel: 1, // Label par défaut
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
    },

    updateActiveLabel(newLabel) {
      // Met à jour le label actif lorsque MosaicView envoie un nouveau label
      this.activeLabel = newLabel;
    },
  }
};
</script>

<style scoped>
#app {
  text-align: center;
}

.view-toggle {
  display: flex;
  align-items: center;
  gap: 10px;
  justify-content: center;
  margin: 20px 0;
}

.switch {
  position: relative;
  display: inline-block;
  width: 50px;
  height: 25px;
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: 0.4s;
  border-radius: 25px;
}

.slider:before {
  position: absolute;
  content: "";
  height: 18px;
  width: 18px;
  left: 4px;
  bottom: 3.5px;
  background-color: white;
  transition: 0.4s;
  border-radius: 50%;
}

input:checked + .slider {
  background-color: #4caf50;
}

input:checked + .slider:before {
  transform: translateX(24px);
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
