<template>
  <div class="video-player-container">
    <input type="file" @change="loadVideo" accept="video/*" />
    <video
      ref="video"
      :src="videoSource"
      @timeupdate="updateCurrentTime"
      @loadedmetadata="onVideoLoaded"
      class="video-player"
      muted
    ></video>
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
  },
};
</script>

<style scoped>
.video-player-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px;
}
.video-player {
  width: 100%;
  max-width: 800px;
  height: auto;
  background: black;
}
</style>
