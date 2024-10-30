<template>
  <div class="mosaic-container">
    <!-- Upload Button -->
    <div v-if="!videoFile" class="upload-section">
      <input type="file" @change="handleVideoUpload" accept="video/*" />
      <p>Upload a video to display in segments</p>
    </div>

    <!-- Mosaic Video Grid -->
    <div v-else class="mosaic-preview">
      <div class="video-grid">
        <div
          v-for="(segment, index) in segments"
          :key="index"
          class="video-section"
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
      <button @click="resetVideo" class="reset-button">Upload New Video</button>
    </div>

    <div v-if="errorMessage" class="error-message">{{ errorMessage }}</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      videoFile: null,        // Holds the uploaded video file
      videoUrl: null,         // URL for the video file to display
      errorMessage: "",       // Error message display
      segments: [],           // Holds segment time ranges
      segmentDuration: 3,     // Segment duration in seconds (adjust as needed)
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

        // Initialize segments based on duration after metadata is loaded
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
          endTime: Math.min(startTime + this.segmentDuration, duration), // End time cannot exceed the video's duration
        });
      }
    },
    setSegmentStart(event, startTime) {
      const videoElement = event.target;
      videoElement.currentTime = startTime;

      // Restart at the specified start time when video loops
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

.mosaic-preview {
  margin-top: 20px;
}

.video-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 3px;
  justify-items: center;
  max-width: 80%;
}

.video-section {
  display: flex;
  align-items: center;
  justify-content: center;
  border: 2px solid #5e5e5e; /* Adjust color as needed */
  border-radius: 8px;
  overflow: hidden;
}

.video-player {
  width: 100%;
  height: auto;
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
