<template>
  <div class="mosaic-container" @keydown="handleKeydown" tabindex="0">
    <!-- Affichage des informations générales de la vidéo -->
    <div v-if="videoInfo" class="video-info">
      <p><strong>Original Video Path:</strong> <a :href="videoInfo.original_video_path" target="_blank">{{ videoInfo.original_video_path }}</a></p>
      <p><strong>Compressed Video Path:</strong> <a :href="videoInfo.compressed_video_path" target="_blank">{{ videoInfo.compressed_video_path }}</a></p>
      <p><strong>Scene Data JSON Path:</strong> <a :href="videoInfo.scene_data_path" target="_blank">{{ videoInfo.scene_data_path }}</a></p>
      <p><strong>Upload Date:</strong> {{ videoInfo.upload_date }}</p>
      <p><strong>Selected Sport:</strong> {{ videoInfo.selectedSport }}</p>
    </div>

    <!-- Mosaic Content -->
    <div class="mosaic-content">
      <!-- Video Grid Section -->
      <div class="video-grid-container" @wheel="handleScroll">
        <div class="video-grid" :style="{ '--num-rows': numRows }">
          <div
            v-for="(clip, index) in filteredClips"
            :key="index"
            :class="['video-section', { selected: selectedVideos.includes(index) }]"
            :style="{ borderColor: getLabelColor(index), borderWidth: labeledVideos[index] ? '4px' : '3px' }"
            @mouseover="hoverVideo(index)"
            @mouseleave="hoverVideo(null)"
            @click="handleClick(index)"
          >
            <video
              :src="videoInfo.compressed_video_path"
              autoplay
              muted
              loop
              class="video-player"
              @loadedmetadata="setSegmentStart($event, clip.startTime)"
              @timeupdate="loopSegment($event, clip.endTime)"
            ></video>
            <span v-if="labeledVideos[index]" 
                  class="label-text"
                  :style="{ backgroundColor: getLabelColor(index) }">
              {{ labeledVideos[index] }}
            </span>
          </div>
        </div>
      </div>

      <!-- Video Preview Section -->
      <div class="video-preview">
        <video
          v-if="hoveredVideo !== null"
          :src="videoInfo.compressed_video_path"
          autoplay
          muted
          loop
          :currentTime="filteredClips[hoveredVideo]?.startTime"
          class="preview-player"
        ></video>
        <p v-else class="preview-placeholder">Survolez une vidéo pour la prévisualiser</p>
      </div>
    </div>
  </div>
</template>

<script>
import sportsConfigurations from "@/assets/sportsConfigurations.js";
import colors from "@/assets/colors.js";

export default {
  props: {
    videoInfo: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      hoveredVideo: null,
      selectedVideos: [],
      labeledVideos: {}, // Store labels by video index
      sceneData: [],
      filteredClips: [],
      availableSports: [],
      numRows: 9, // Nombre de lignes dans la grille
      currentLabelIndex: 0 // Index du label actuellement sélectionné
    };
  },
  computed: {
    currentLabel() {
      const sportConfig = sportsConfigurations[this.videoInfo.selectedSport];
      if (sportConfig && sportConfig.events && sportConfig.events.length > 0) {
        return sportConfig.events[this.currentLabelIndex];
      } else {
        console.warn(`Aucun événement trouvé pour le sport sélectionné : ${this.videoInfo.selectedSport}`);
        return null;
      }
    }
  },
  async created() {
    await this.loadSceneData();
  },
  methods: {
    async loadSceneData() {
      if (!this.videoInfo.scene_data_path) {
        console.error("Le chemin des données de scène (scene_data_path) est manquant.");
        return;
      }

      try {
        const response = await fetch(this.videoInfo.scene_data_path);
        if (!response.ok) {
          throw new Error(`Erreur HTTP : ${response.status}`);
        }
        const data = await response.json();
        this.sceneData = data.scenes;
        this.initializeAvailableSports(data.scenes);
        this.generateClips();
      } catch (error) {
        console.error("Erreur lors du chargement des données de scène:", error);
      }
    },
    initializeAvailableSports(data) {
      const sports = new Set(data.map(scene => scene.recognized_sport));
      this.availableSports = Array.from(sports);
    },
    generateClips() {
      const sportScenes = this.sceneData.filter(
        scene => scene.recognized_sport === this.videoInfo.selectedSport
      );

      this.filteredClips = [];
      sportScenes.forEach(scene => {
        const startTime = this.timeToSeconds(scene.start);
        const endTime = this.timeToSeconds(scene.end);

        for (let t = startTime; t < endTime; t += 1) {
          this.filteredClips.push({
            startTime: t,
            endTime: Math.min(t + 1, endTime)
          });
        }
      });
    },
    timeToSeconds(timeStr) {
      const [hours, minutes, seconds] = timeStr.split(":");
      return (
        parseInt(hours, 10) * 3600 +
        parseInt(minutes, 10) * 60 +
        parseFloat(seconds)
      );
    },
    setSegmentStart(event, startTime) {
      event.target.currentTime = startTime;
    },
    loopSegment(event, endTime) {
      if (event.target.currentTime >= endTime) {
        event.target.currentTime = endTime - 1;
      }
    },
    handleScroll(event) {
      const container = event.currentTarget;
      container.scrollLeft += event.deltaY;
      event.preventDefault();
    },
    hoverVideo(index) {
      this.hoveredVideo = index;
    },
    handleClick(index) {
      this.labelVideo(index); // Ajoute ou retire un label au clic
    },
    labelVideo(index) {
      const label = this.currentLabel;
      if (!label) {
        console.error("Impossible d'étiqueter la vidéo : aucun label actuel n'est défini.");
        return;
      }
      
      // Vérifie si la case possède déjà ce label
      if (this.labeledVideos[index] === label) {
        // Si oui, supprime le label
        delete this.labeledVideos[index];
        console.log(`Label supprimé pour l'index ${index}`);
      } else {
        // Sinon, attribue le label
        this.labeledVideos[index] = label;
        console.log(`Label attribué : ${label} pour l'index ${index}`);
      }
    },
    handleKeydown(event) {
      const key = parseInt(event.key);
      const sportConfig = sportsConfigurations[this.videoInfo.selectedSport];
      if (key > 0 && sportConfig && key <= sportConfig.events.length) {
        this.currentLabelIndex = key - 1; // Change l'index du label
        console.log("Label changé à :", this.currentLabel);
      }
    },
    getLabelColor(index) {
      const label = this.labeledVideos[index];
      if (!label) return '#5e5e5e'; // Couleur par défaut si aucun label n'est attribué
      const sportConfig = sportsConfigurations[this.videoInfo.selectedSport];
      const eventIndex = sportConfig.events.indexOf(label);
      return colors[eventIndex % colors.length]; // Associe une couleur basée sur l'index de l'événement
    }
  }
};
</script>

<style scoped>
.mosaic-container {
  text-align: center;
  padding: 10px;
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
  border-width: 4px;
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

.label-text {
  position: absolute;
  bottom: 4px;
  left: 4px;
  font-size: 12px;
  /* font-weight: bold; */
  color: #fff;
  background-color: rgba(0, 0, 0, 0.5);
  padding: 2px 4px;
  border-radius: 4px;
}
</style>
