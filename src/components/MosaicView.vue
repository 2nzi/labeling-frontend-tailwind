<template>
  <div class="mosaic-container" @keydown="handleKeydown" tabindex="0">
    <!-- Loading Section -->
    <div v-if="loading" class="loading-section">
      <p>Compression en cours, veuillez patienter...</p>
      <div class="loader"></div>
    </div>

    <!-- Upload Section -->
    <div v-else-if="!videoFile" class="upload-section">
      <!-- Custom Drag-and-Drop Upload Zone -->
      <label
        class="drop-zone"
        @dragover.prevent
        @drop.prevent="handleDrop"
      >
        <input type="file" id="file-upload" @change="handleVideoUpload" accept="video/*" class="file-input"/>
        <div class="drop-zone-content">
          <i class="upload-icon"></i>
          <p class="drop-zone-title">DÉPOSEZ VOTRE FICHIER ICI</p>
          <p class="drop-zone-subtitle">Glissez-déposez votre vidéo ou cliquez pour sélectionner un fichier</p>
        </div>
      </label>
    </div>

    <!-- Mosaic Content -->
    <div v-else class="mosaic-content">
      <!-- Video Grid Section -->
      <div class="video-grid-container" :style="{ '--num-rows': numRows }" @wheel="handleScroll">
        <div class="video-grid">
          <div
            v-for="(segment, index) in segments"
            :key="index"
            :class="['video-section', { selected: selectedVideos.includes(index) }]"
            :style="{ borderColor: labeledVideos[index] ? getLabelColor(labeledVideos[index]) : '#5e5e5e' }"
            @mouseover="hoverVideo(index)"
            @mouseleave="hoverVideo(null)"
            @click="handleClick(index, $event)"
            >
            <video
              :src="compressedVideoUrl"
              autoplay
              muted
              loop
              :currentTime="segment.startTime"
              @loadedmetadata="setSegmentStart($event, segment.startTime)"
              class="video-player"
            ></video>
            <span v-if="labeledVideos[index]" 
                  :style="{ color: getLabelColor(labeledVideos[index]) }" 
                  class="label-text">
              {{ labeledVideos[index] }}
            </span>
          </div>
        </div>
      </div>

      <!-- Video Preview Section -->
      <div class="video-preview">
        <video
          v-if="hoveredVideo !== null"
          :src="compressedVideoUrl"
          autoplay
          muted
          loop
          :currentTime="segments[hoveredVideo].startTime"
          key="preview-video"
          class="preview-player"
        ></video>
        <p v-else class="preview-placeholder">Hover over a video to preview</p>
      </div>
    </div>

    <!-- Reset Button -->
    <button @click="resetVideo" class="reset-button">Charger une nouvelle vidéo</button>
    <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
  </div>
</template>

<script>
import sportsConfigurations from "@/assets/sportsConfigurations.js";
import colors from "@/assets/colors.js";

export default {
  data() {
    return {
      videoFile: null,
      compressedVideoUrl: null,
      errorMessage: "",
      segments: [],
      segmentDuration: 2,
      selectedVideos: [], // Store multiple selected video indices
      hoveredVideo: null,
      loading: false,
      numRows: 9,
      currentSport: "Surf", // Specify the sport type here
      currentLabelIndex: 0, // Active label index
      labeledVideos: {}, // Store labels by video index
    };
  },
  computed: {
    currentLabel() {
      return sportsConfigurations[this.currentSport].events[this.currentLabelIndex];
    },
  },

  methods: {
    handleClick(index, event) {
    if (event.ctrlKey) {
      this.selectVideo(index, event);
    } else {
      this.labelVideo(index);
    }
  },
    handleVideoUpload(event) {
      const file = event.target.files[0];
      this.processUpload(file);
    },
    handleDrop(event) {
      const file = event.dataTransfer.files[0];
      this.processUpload(file);
    },
    processUpload(file) {
      if (file && file.type.startsWith("video/")) {
        this.videoFile = file;
        this.loading = true;
        this.uploadAndCompressVideo(file);
      } else {
        this.errorMessage = "Veuillez déposer un fichier vidéo valide.";
      }
    },
    async uploadAndCompressVideo(file) {
      const formData = new FormData();
      formData.append("file", file);

      try {
        const response = await fetch("http://localhost:8000/upload-video", {
          method: "POST",
          body: formData,
        });

        if (response.ok) {
          const data = await response.json();
          this.compressedVideoUrl = data.compressed_video_path;
          this.loading = false;

          const video = document.createElement("video");
          video.src = this.compressedVideoUrl;
          video.onloadedmetadata = () => {
            const duration = video.duration;
            this.initializeSegments(duration);
          };
        } else {
          this.errorMessage = "Erreur lors de la compression de la vidéo.";
          this.loading = false;
        }
      } catch (error) {
        this.errorMessage = "Échec de la compression.";
        this.loading = false;
      }
    },
    initializeSegments(duration) {
      this.segments = [];
      for (let startTime = 0; startTime < duration; startTime += this.segmentDuration) {
        this.segments.push({
          startTime: startTime,
          endTime: Math.min(startTime + this.segmentDuration, duration),
        });
      }
    },
    setSegmentStart(event, startTime) {
      const videoElement = event.target;
      videoElement.currentTime = startTime;

      videoElement.addEventListener("timeupdate", () => {
        if (videoElement.currentTime >= startTime + this.segmentDuration || videoElement.currentTime >= videoElement.duration) {
          videoElement.currentTime = startTime;
        }
      });
    },
    resetVideo() {
      this.videoFile = null;
      this.compressedVideoUrl = null;
      this.segments = [];
      this.selectedVideo = null;
      this.hoveredVideo = null;
      this.loading = false;
      this.labeledVideos = {}; // Clear labels on reset
    },
    handleScroll(event) {
      const container = event.currentTarget;
      container.scrollLeft += event.deltaY;
      event.preventDefault();
    },
    labelVideo(index) {
      const label = this.currentLabel;
      const color = this.getLabelColor(label);
      this.labeledVideos[index] = label;
      this.selectedVideo = index;
      
      // Log de l'événement et de la couleur
      console.log(`Label attribué : ${label}`);
      console.log(`Couleur : ${color}`);
    },
    hoverVideo(index) {
      this.hoveredVideo = index;
    },
    selectVideo(index) {
      // Toggle selection when Ctrl is pressed
      if (this.selectedVideos.includes(index)) {
        this.selectedVideos = this.selectedVideos.filter((i) => i !== index);
      } else {
        this.selectedVideos.push(index);
      }
      console.log(`Selected videos:`, this.selectedVideos);
    },
    handleKeydown(event) {
      if (event.key === "j" && this.selectedVideos.length > 1) {
        this.joinSelectedVideos();
      }
      const key = parseInt(event.key);
      if (key > 0 && key <= sportsConfigurations[this.currentSport].events.length) {
        this.currentLabelIndex = key - 1;
      }
    },
    joinSelectedVideos() {
      // Sort selected videos by their indices
      this.selectedVideos.sort((a, b) => a - b);

      // Get start and end times from the first and last selected segments
      const startTime = this.segments[this.selectedVideos[0]].startTime;
      const endTime = this.segments[this.selectedVideos[this.selectedVideos.length - 1]].endTime;

      // Create a new joined segment
      const newSegment = { startTime, endTime };
      
      // Update segments by replacing selected segments with the new joined segment
      this.segments = [
        ...this.segments.slice(0, this.selectedVideos[0]),
        newSegment,
        ...this.segments.slice(this.selectedVideos[this.selectedVideos.length - 1] + 1),
      ];

      // Clear selected videos after joining
      this.selectedVideos = [];
      console.log(`Joined videos into new segment from ${startTime}s to ${endTime}s`);
    },

    getLabelColor(label) {
      const eventIndex = sportsConfigurations[this.currentSport].events.indexOf(label);
      return colors[eventIndex % colors.length]; // Utilise la couleur correspondante dans le tableau
    },

  },
  mounted() {
    this.$el.focus();
  },
};
</script>

<style scoped>
:root {
  --num-rows: 5;
}

.mosaic-container {
  text-align: center;
  padding: 10px;
}

.upload-section {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.drop-zone {
  width: 300px;
  height: 200px;
  border: 2px dashed #007acc;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.drop-zone:hover {
  background-color: #f3faff;
}

.file-input {
  display: none;
}

.drop-zone-content {
  text-align: center;
  color: #555;
}

.drop-zone-title {
  font-weight: bold;
  font-size: 16px;
  color: #333;
}

.drop-zone-subtitle {
  font-size: 12px;
  color: #888;
}

.loading-section {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.loader {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 2s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.mosaic-content {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  gap: 20px;
  margin-top: 20px;
  max-height: 80vh;
}

.video-grid-container {
  width: 60%;
  overflow-x: auto;
  max-height: 80vh;
  white-space: nowrap;
}

.video-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  grid-auto-flow: column;
  grid-template-rows: repeat(var(--num-rows), 1fr);
  gap: 3px;
}

.video-section {
  position: relative;
  width: 150px;
  aspect-ratio: 16 / 9;
  border: 3px solid #5e5e5e;
  border-radius: 8px;
  overflow: hidden;
  cursor: pointer;
}

.video-section.selected {
  border: 4px solid #f30101;
}

.video-player {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.video-preview {
  width: 35%;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #ccc;
  border-radius: 8px;
  height: 80vh;
}

.preview-player {
  width: 100%;
  height: auto;
}

.preview-placeholder {
  color: #888;
  font-size: 1.2em;
  text-align: center;
}

.reset-button {
  margin-top: 20px;
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

.reset-button:hover {
  background: linear-gradient(135deg, #007acc, #005ea0);
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
}

.error-message {
  color: red;
  margin-top: 10px;
}

.label-text {
  position: absolute;
  bottom: 4px; /* Adjust to place it within the video */
  left: 4px;
  font-size: 12px;
  font-weight: bold;
  /* background-color: rgba(255, 255, 255, 0.7); */
  padding: 2px 4px;
  border-radius: 4px;
}

</style>
