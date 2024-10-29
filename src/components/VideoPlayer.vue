<template>
    <div
      class="video-player-container"
      :class="{ empty: !videoSource }"
      @click="triggerFileUpload"
    >
      <!-- Input caché pour charger la vidéo -->
      <input type="file" ref="fileInput" @change="loadVideo" accept="video/*" style="display: none;" />
      
      <!-- Lecteur vidéo, visible uniquement quand la vidéo est chargée -->
      <video
        v-if="videoSource"
        ref="video"
        :src="videoSource"
        @timeupdate="updateCurrentTime"
        @loadedmetadata="onVideoLoaded"
        class="video-player"
        muted
      ></video>
  
      <!-- Placeholder, visible uniquement quand aucune vidéo n'est chargée -->
      <div v-else class="placeholder">
        Upload
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: {
      currentTime: Number,
      isPlaying: Boolean,
    },
    data() {
      return {
        videoSource: null,
      };
    },
    watch: {
      currentTime(newTime) {
        if (this.$refs.video && Math.abs(this.$refs.video.currentTime - newTime) > 0.1) {
          this.$refs.video.currentTime = newTime;
        }
      },
      isPlaying(newPlayingState) {
        if (this.$refs.video) {
          if (newPlayingState) {
            this.$refs.video.play();
          } else {
            this.$refs.video.pause();
          }
        }
      },
    },
    methods: {
      loadVideo(event) {
        const file = event.target.files[0];
        if (file) {
          this.videoSource = URL.createObjectURL(file);
          this.$emit("videoUploaded");
        }
      },
      updateCurrentTime() {
        const currentTime = this.$refs.video.currentTime;
        this.$emit("updateCurrentTime", currentTime);
      },
      onVideoLoaded() {
        const videoDuration = this.$refs.video.duration;
        this.$emit("videoDuration", videoDuration);
      },
      togglePlayback() {
        if (this.isPlaying) {
          this.$refs.video.pause();
        } else {
          this.$refs.video.play();
        }
      },
      triggerFileUpload() {
        if (!this.videoSource) {
          this.$refs.fileInput.click();
        }
      },
    },
  };
  </script>
  
  <style scoped>
.video-player-container {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 80%;
  height: 450px;
  background: rgb(152, 152, 152);
  cursor: pointer;
  position: relative;
}

@media (min-width: 900px) {
  .video-player-container {
    min-width: 800px;
  }
}
  
  /* .video-player-container.empty {
    border: 2px dashed #aaa;
  } */
  
  .video-player {
    width: 100%;
    height: 100%;
  }
  
  .placeholder {
    color: #ccc;
    font-size: 1.5em;
    text-align: center;
  }
  </style>
  