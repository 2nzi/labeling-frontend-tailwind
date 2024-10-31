<template>
  <div class="mosaic-container">
    <!-- Upload Button -->
    <div v-if="!videoFile" class="upload-section">
      <input type="file" @change="handleVideoUpload" accept="video/*" />
      <p>Upload a video to display in segments</p>
    </div>

    <!-- Mosaic View with Grid and Preview Section -->
    <div v-else class="mosaic-content">
      <!-- Left: Video Grid with Horizontal Scroll -->
      <div class="video-grid-container" @wheel="handleScroll">
        <div class="video-grid">
          <div
            v-for="(segment, index) in segments"
            :key="index"
            :class="['video-section', { selected: selectedVideo === index }]"
            @mouseover="hoverVideo(index)"
            @mouseleave="hoverVideo(null)"
            @click="selectVideo(index)"
          >
            <video
              :src="videoUrl"
              autoplay
              muted
              loop
              :currentTime="segment.startTime"
              @loadedmetadata="setSegmentStart($event, segment.startTime)"
              class="video-player"
            ></video>
          </div>
        </div>
      </div>

      <!-- Right: Preview of Hovered Video -->
      <div class="video-preview">
        <video
          v-if="hoveredVideo !== null"
          :src="videoUrl"
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

    <button @click="resetVideo" class="reset-button">Upload New Video</button>

    <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      videoFile: null,
      videoUrl: null,
      errorMessage: "",
      segments: [],
      segmentDuration: 3,
      selectedVideo: null, // State to track the selected video
      hoveredVideo: null,   // State to track the hovered video for preview
    };
  },
  methods: {
    handleVideoUpload(event) {
      const file = event.target.files[0];
      if (file) {
        if (!file.type.startsWith("video/")) {
          this.errorMessage = "Please upload a valid video file.";
          return;
        }

        this.videoFile = file;
        this.videoUrl = URL.createObjectURL(file);
        this.errorMessage = "";

        const video = document.createElement("video");
        video.src = this.videoUrl;
        video.onloadedmetadata = () => {
          const duration = video.duration;
          this.initializeSegments(duration);
        };
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
      this.videoUrl = null;
      this.segments = [];
      this.selectedVideo = null;
      this.hoveredVideo = null;
    },
    handleScroll(event) {
      const container = event.currentTarget;
      container.scrollLeft += event.deltaY;
      event.preventDefault();
    },
    selectVideo(index) {
      this.selectedVideo = index;
    },
    hoverVideo(index) {
      this.hoveredVideo = index;
    },
  },
};
</script>

<style scoped>
.mosaic-container {
  text-align: center;
  padding: 10px;
}

.upload-section {
  margin-top: 20px;
}

/* Layout of grid and preview sections */
.mosaic-content {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  gap: 20px;
  margin-top: 20px;
  max-height: 80vh; /* Limit the height of the mosaic content to 80% of viewport */
}

/* Left: Scrollable Grid */
.video-grid-container {
  width: 60%;
  overflow-x: auto;
  overflow-y: hidden;
  max-height: 80vh; /* Limit the grid container's height */
  white-space: nowrap;
}

.video-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  grid-auto-flow: column;
  grid-template-rows: repeat(5, 1fr);
  gap: 3px;
  align-items: start;
  padding: 0;
}

.video-section {
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #5e5e5e;
  border-radius: 8px;
  overflow: hidden;
  width: 150px;
  height: calc(80vh / 5 - 6px); /* Divide 80% height by 5 rows, adjusting for gaps */
  cursor: pointer;
}

.video-section.selected {
  border: 3px solid #f30101; /* Red border for selected video */
}

.video-player {
  width: 100%;
  height: 100%;
}

/* Right: Preview of hovered video */
.video-preview {
  width: 35%;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #ccc;
  border-radius: 8px;
  height: 80vh; /* Match height limit of the grid */
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
  margin-top: 10px;
  background-color: #007acc;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.reset-button:hover {
  background-color: #005ea0;
}

.error-message {
  color: red;
  margin-top: 10px;
}
</style>
