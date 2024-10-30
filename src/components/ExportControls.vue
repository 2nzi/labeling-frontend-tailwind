<template>
    <div class="export-controls">
      <button @click="exportAsJSON" class="export-button">Exporter en JSON</button>
      <button @click="exportAsZip" class="export-button">Exporter en ZIP (découpage vidéo)</button>
    </div>
  </template>
  
  <script>
  import JSZip from "jszip";
  import { saveAs } from "file-saver";
  
  export default {
    props: ["events", "videoElement"],
    methods: {
      async exportAsJSON() {
        const exportData = this.events.map(event => ({
          event: event.name,
          blocks: event.blocks.map(block => ({
            id: block.name,
            start: block.start,
            end: block.start + block.duration
          }))
        }));
        
        const jsonBlob = new Blob([JSON.stringify(exportData, null, 2)], { type: "application/json" });
        saveAs(jsonBlob, "export_data.json");
      },
  
      async exportAsZip() {
        const zip = new JSZip();
        const videoElement = this.videoElement;
  
        const exportData = this.events.map(event => ({
          event: event.name,
          blocks: event.blocks.map(block => ({
            id: block.name,
            start: block.start,
            end: block.start + block.duration
          }))
        }));
  
        for (const event of exportData) {
          const folder = zip.folder(event.event);
  
          for (const block of event.blocks) {
            const videoBlob = await this.cutVideoBlob(videoElement, block.start, block.end);
            const fileName = `block_${block.id}.mp4`;
            folder.file(fileName, videoBlob);
          }
        }
  
        const zipBlob = await zip.generateAsync({ type: "blob" });
        saveAs(zipBlob, "videos_decoupees.zip");
      },
  
      cutVideoBlob(videoElement, start, end) {
        return new Promise(resolve => {
          const mediaRecorder = new MediaRecorder(videoElement.captureStream());
          const chunks = [];
  
          videoElement.currentTime = start;
          videoElement.play();
  
          mediaRecorder.ondataavailable = event => chunks.push(event.data);
          mediaRecorder.onstop = () => {
            const blob = new Blob(chunks, { type: "video/mp4" });
            resolve(blob);
          };
          mediaRecorder.start();
  
          videoElement.ontimeupdate = () => {
            if (videoElement.currentTime >= end) {
              videoElement.pause();
              mediaRecorder.stop();
              videoElement.ontimeupdate = null;
            }
          };
        });
      }
    }
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
  }
  .export-button:hover {
    background-color: #45a049;
  }
  </style>
  