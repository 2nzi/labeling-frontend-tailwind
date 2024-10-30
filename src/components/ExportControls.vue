<template>
  <div class="export-controls">
    <button @click="exportAsJSON" class="export-button" :disabled="!isVideoLoaded">Download JSON</button>
    <button @click="exportAsZip" class="export-button" :disabled="!isVideoLoaded">Download Video ZIP</button>
  </div>
</template>

<script>
import JSZip from "jszip";
import { saveAs } from "file-saver";

export default {
  props: ["events", "isVideoLoaded", "cutVideoBlob"],  // Passing in cutVideoBlob
  methods: {
    exportAsJSON() {
      const exportData = this.events.map(event => ({
        event: event.name,
        blocks: event.blocks.map(block => ({
          id: block.name,
          start: block.start,
          end: block.start + block.duration,
        })),
      }));

      const jsonBlob = new Blob([JSON.stringify(exportData, null, 2)], { type: "application/json" });
      saveAs(jsonBlob, "export_data.json");
    },

    async exportAsZip() {
      const zip = new JSZip();

      for (const event of this.events) {
        const folder = zip.folder(event.name);

        for (const block of event.blocks) {
          const start = block.start;
          const end = block.start + block.duration;

          try {
            const videoBlob = await this.cutVideoBlob(start, end);  // Use the passed cutVideoBlob method
            if (videoBlob && videoBlob.size > 0) {
              const fileName = `block_${block.name}.mp4`;
              folder.file(fileName, videoBlob);
            } else {
              console.warn(`Block ${block.name} resulted in an empty blob.`);
            }
          } catch (error) {
            console.error(`Failed to export video block ${block.name}:`, error);
          }
        }
      }

      const zipBlob = await zip.generateAsync({ type: "blob" });
      saveAs(zipBlob, "videos_decoupees.zip");
    },
  },
};
</script>

<style scoped>
.export-controls {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}
.export-button {
  background-color: #4CAF50;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}
.export-button:hover {
  background-color: #45a049;
}
.export-button:disabled {
  background-color: #d3d3d3;
  color: #a0a0a0;
  cursor: not-allowed;
}
</style>
