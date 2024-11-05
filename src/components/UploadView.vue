<template>
  <div class="upload-container">
    <h2>Upload Your Video</h2>

    <!-- Sélection du sport avec styles -->
    <label for="sport-select" class="sport-select-label">Select Sport:</label>
    <select @change="sportSelected($event)" id="sport-select" class="sport-select">
      <option v-for="(config, sport) in sportsConfigurations" :key="sport" :value="sport">
        {{ sport }}
      </option>
    </select>

    <label class="drop-zone" @dragover.prevent @drop.prevent="handleDrop">
      <input type="file" @change="handleVideoUpload" accept="video/*" class="file-input" />
      <div class="drop-zone-content">
        <p>Drag & drop your video here or click to select</p>
      </div>
    </label>

    <div v-if="loading" class="progress">
      Uploading... {{ progress }}%
    </div>
    <p v-if="errorMessage" class="error">{{ errorMessage }}</p>

    <!-- Affichage des informations de la vidéo et des scènes si disponibles -->
    <div v-if="videoInfo" class="video-info">
      <p><strong>Video URL:</strong> <a :href="videoInfo.original_video_path" target="_blank">{{ videoInfo.original_video_path }}</a></p>
      <p><strong>Compressed Video URL:</strong> <a :href="videoInfo.compressed_video_path" target="_blank">{{ videoInfo.compressed_video_path }}</a></p>
      <p><strong>Scene JSON URL:</strong> <a :href="videoInfo.scene_data_path" target="_blank">{{ videoInfo.scene_data_path }}</a></p>
    </div>
  </div>
</template>

<script>
import sportsConfigurations from "@/assets/sportsConfigurations.js";

export default {
  data() {
    return {
      loading: false,
      progress: 0,
      errorMessage: "",
      videoInfo: null, // Pour stocker les informations de la vidéo
      selectedSport: "Surfing", // Sport par défaut
      sportsConfigurations,
    };
  },
  methods: {
    sportSelected(event) {
      this.selectedSport = event.target.value;
    },
    handleVideoUpload(event) {
      const file = event.target.files[0];
      if (file && file.type.startsWith("video/")) {
        this.uploadVideo(file);
      } else {
        this.errorMessage = "Please select a valid video file.";
      }
    },
    handleDrop(event) {
      const file = event.dataTransfer.files[0];
      if (file && file.type.startsWith("video/")) {
        this.uploadVideo(file);
      } else {
        this.errorMessage = "Please select a valid video file.";
      }
    },
    async uploadVideo(file) {
      this.loading = true;
      this.errorMessage = "";

      const formData = new FormData();
      formData.append("file", file);

      try {
        const response = await fetch("http://localhost:8000/upload-video", {
          method: "POST",
          body: formData,
        });

        const data = await response.json();
        if (response.ok) {
          console.log("Upload successful, response data:", data);

          // Inclure le sport sélectionné dans l'information envoyée
          this.videoInfo = {
            original_video_path: data.original_video_path,
            compressed_video_path: data.compressed_video_path,
            scene_data_path: data.scene_data_path,
            selectedSport: this.selectedSport // Transmettre le sport sélectionné
          };

          // Émettre l'événement avec les informations de la vidéo et du sport
          this.$emit("videoUploaded", this.videoInfo);
        } else {
          this.errorMessage = data.message || "Upload failed.";
        }
      } catch (error) {
        this.errorMessage = "Upload error.";
        console.error("Error uploading video:", error);
      } finally {
        this.loading = false;
      }
    },
  }
};
</script>

<style scoped>
.upload-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.sport-select-label {
  font-size: 1.1em;
  font-weight: 500;
  margin-bottom: 5px;
  color: #333;
}

.sport-select {
  width: 300px;
  padding: 10px;
  font-size: 1em;
  border: 2px solid #007acc;
  border-radius: 8px;
  background-color: #f9f9f9;
  color: #333;
  outline: none;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
  cursor: pointer;
  margin-bottom: 20px;
}

.sport-select:focus {
  border-color: #005ea0;
  box-shadow: 0 0 5px rgba(0, 94, 160, 0.5);
}

.sport-select:hover {
  background-color: #eef6ff;
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
.progress {
  margin-top: 10px;
}
.error {
  color: red;
  margin-top: 10px;
}
.video-info {
  margin-top: 20px;
  text-align: center;
}
</style>
