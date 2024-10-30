<template>
  <div class="mosaic-container">
    <!-- Affiche la zone d'upload si aucune vidéo n'est chargée -->
    <div v-if="!videoUploaded" class="upload-section">
      <input type="file" @change="handleVideoUpload" accept="video/*" />
      <p>Upload a video to generate a 3-segment mosaic preview</p>
    </div>
    
    <!-- Affiche la mosaïque si la vidéo est chargée -->
    <div v-else class="mosaic-grid">
      <div
        v-for="clip in clips"
        :key="clip.id"
        class="clip"
      >
        <video :src="clip.url" controls width="150" height="150" loop muted></video>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      videoUploaded: false, // Statut de l'upload
      clips: [], // Liste des clips à afficher
    };
  },
  methods: {
    async handleVideoUpload(event) {
      const file = event.target.files[0];
      if (file) {
        await this.generateMosaicPreview(file);
      }
    },

    async generateMosaicPreview(videoFile) {
      try {
        // Obtenir la durée de la vidéo
        const duration = await this.getVideoDuration(videoFile);

        // Si la vidéo est suffisamment longue pour 3 segments de 3 secondes
        const clipDuration = 3; // Durée de chaque clip en secondes
        const clipCount = Math.min(3, Math.floor(duration / clipDuration));

        // Effacer les clips précédents
        this.clips = [];

        for (let i = 0; i < clipCount; i++) {
          const clipUrl = await this.createClip(videoFile, i * clipDuration, clipDuration);
          this.clips.push({ id: i, url: clipUrl });
        }

        this.videoUploaded = true;
      } catch (error) {
        console.error("Erreur lors de la génération des clips :", error);
      }
    },

    async getVideoDuration(file) {
      return new Promise((resolve) => {
        const video = document.createElement("video");
        video.preload = "metadata";
        video.src = URL.createObjectURL(file);
        video.onloadedmetadata = () => {
          resolve(video.duration);
          URL.revokeObjectURL(video.src);
        };
      });
    },

    async createClip(videoFile, startTime) {
      return new Promise((resolve) => {
        const videoUrl = URL.createObjectURL(videoFile);
        const videoElement = document.createElement("video");
        videoElement.src = videoUrl;
        videoElement.currentTime = startTime;

        // Créer une boucle de lecture de "duration" secondes en utilisant `onseeked`
        videoElement.onseeked = () => {
          videoElement.pause();
          resolve(videoUrl);
        };

        videoElement.play();
      });
    },
  },
};
</script>

<style scoped>
.mosaic-container {
  text-align: center;
}
.upload-section {
  margin-top: 20px;
}
.mosaic-grid {
  display: flex;
  gap: 10px;
  justify-content: center;
}
.clip {
  width: 150px;
  height: 150px;
  overflow: hidden;
}
</style>
