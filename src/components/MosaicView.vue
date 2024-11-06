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
            :style="getClipStyle(index)"
            @mouseover="hoverVideo(index)"
            @mouseleave="hoverVideo(null)"
            @click="handleClick(index, $event)"
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
    <SaveLabelButton
      :generateLabelingData="generateLabelingData"
      @save-success="onSaveSuccess"
      @save-error="onSaveError"
      />
  </div>
</template>

<script>
import sportsConfigurations from "@/assets/sportsConfigurations.js";
import colors from "@/assets/colors.js";
import SaveLabelButton from "@/components/SaveLabelButton.vue";

export default {
  components: {
    SaveLabelButton
  },
  props: {
    videoInfo: {
      type: Object,
      required: true
    }
  },
  data() {
    return {
      hoveredVideo: null,
      selectedVideos: [], // Array to store indices of selected clips
      labeledVideos: {}, // Store labels by video index
      joinedGroups: [], // Array to store joined groups of indices
      sceneData: [],
      filteredClips: [],
      availableSports: [],
      numRows: 9, // Number of rows in the grid
      currentLabelIndex: 0 // Currently selected label index
    };
  },
  computed: {
    currentLabel() {
      const sportConfig = sportsConfigurations[this.videoInfo.selectedSport];
      if (sportConfig && sportConfig.events && sportConfig.events.length > 0) {
        return sportConfig.events[this.currentLabelIndex];
      } else {
        console.warn(`No events found for the selected sport: ${this.videoInfo.selectedSport}`);
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
        console.error("Scene data path (scene_data_path) is missing.");
        return;
      }

      try {
        const response = await fetch(this.videoInfo.scene_data_path);
        if (!response.ok) {
          throw new Error(`HTTP error: ${response.status}`);
        }
        const data = await response.json();
        this.sceneData = data.scenes;
        this.initializeAvailableSports(data.scenes);
        this.generateClips();
      } catch (error) {
        console.error("Error loading scene data:", error);
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
    handleClick(index, event) {
      // Check if Ctrl is pressed for multi-selection
      if (event.ctrlKey) {
        if (this.selectedVideos.includes(index)) {
          // If the index is already selected, remove it
          this.selectedVideos = this.selectedVideos.filter(i => i !== index);
        } else {
          // Otherwise, add it to the selection
          this.selectedVideos.push(index);
        }
      } else {
        // Reset multi-selection if Ctrl is not pressed
        this.selectedVideos = [index];
      }

      // Apply label directly if only one index is selected
      if (this.selectedVideos.length === 1) {
        this.labelVideo(index);
      }
    },
    
    labelVideo(index) {
      const label = this.currentLabel;
      if (!label) {
        console.error("Cannot label video: no current label is defined.");
        return;
      }
      // Toggle label
      if (this.labeledVideos[index] === label) {
        delete this.labeledVideos[index];
      } else {
        this.labeledVideos[index] = label;
      }
    },
    handleKeydown(event) {
      const key = parseInt(event.key);
      const sportConfig = sportsConfigurations[this.videoInfo.selectedSport];
      if (key > 0 && sportConfig && key <= sportConfig.events.length) {
        this.currentLabelIndex = key - 1;
      }
      // Join selected videos on pressing 'J'
      if (event.key === 'j' && this.selectedVideos.length > 1) {
        this.joinSelectedVideos();
      }
    },
    joinSelectedVideos() {
      // Check if a group with selected indices already exists
      const existingGroupIndex = this.joinedGroups.findIndex(group =>
        group.length === this.selectedVideos.length &&
        group.every(idx => this.selectedVideos.includes(idx))
      );

      if (existingGroupIndex > -1) {
        // If group exists, remove it
        this.joinedGroups.splice(existingGroupIndex, 1);
        console.log("Joint group removed:", this.selectedVideos);
      } else {
        // If no group exists, create a new group
        this.joinedGroups.push([...this.selectedVideos]);
        console.log("Joint group created:", this.selectedVideos);
      }

      // Clear selection after creating or deleting a group
      this.selectedVideos = [];
    },
    isInJoinedGroup(index) {
      return this.joinedGroups.some(group => group.includes(index));
    },
    getLabelColor(index) {
      const label = this.labeledVideos[index];
      if (!label) return '#5e5e5e'; // Default color if no label
      const sportConfig = sportsConfigurations[this.videoInfo.selectedSport];
      const eventIndex = sportConfig.events.indexOf(label);
      return colors[eventIndex % colors.length];
    },
    getClipStyle(index) {
      const baseColor = this.getLabelColor(index);
      const isSelected = this.selectedVideos.includes(index);
      const isJoined = this.isInJoinedGroup(index);

      return {
        borderColor: baseColor,
        borderWidth: isSelected ? '5px' : '3px',
        outline: isJoined ? '3px solid #F5DF4D' : 'none',
        boxShadow: isJoined
          ? `
              0 0 5px #F5DF4D,
              0 0 10px #F5DF4D,
              0 0 15px #F5DF4D,
              0 0 20px #F5DF4D
            `
          : 'none'
      };
    },
    generateLabelingData() {
    const labeledData = [];
    const duration = 1; // Supposons une durée de 1 seconde par clip
    const videoId = this.videoInfo.scene_data_path
    ? this.videoInfo.scene_data_path.split("/").pop().split("_")[0] // Extrait l'UUID du nom de fichier
    : null;

    Object.entries(this.labeledVideos).forEach(([index, label]) => {
      const start = this.filteredClips[index].startTime;
      const end = this.filteredClips[index].endTime;

      const group = this.joinedGroups.find(g => g.includes(Number(index)));

      if (group) {
        const timestamps = group.map(idx => ({
          id: idx,
          start_time: this.filteredClips[idx].startTime,
          end_time: this.filteredClips[idx].endTime
        }));
        
        labeledData.push({
          label,
          combined_start_time: timestamps[0].start_time,
          combined_end_time: timestamps[timestamps.length - 1].end_time,
          instances: timestamps
        });
      } else {
        labeledData.push({
          label,
          combined_start_time: start,
          combined_end_time: end,
          instances: [{ id: Number(index), start_time: start, end_time: end }]
        });
      }
    });

    // Ajouter les clips non étiquetés sous le label "other"
    const otherInstances = this.filteredClips
      .map((clip, index) => (!this.labeledVideos[index] ? { id: index, start_time: clip.startTime, end_time: clip.endTime } : null))
      .filter(Boolean);

    if (otherInstances.length) {
      labeledData.push({
        label: "other",
        combined_start_time: otherInstances[0].start_time,
        combined_end_time: otherInstances[otherInstances.length - 1].end_time,
        instances: otherInstances
      });
    }

    return {
      metadata: {
        video_id: videoId,
        duration_per_clip: duration
      },
      events: labeledData
    };
  },
  onSaveSuccess() {
    console.log("Données de labellisation sauvegardées avec succès.");
  },
  onSaveError(error) {
    console.error("Erreur lors de la sauvegarde des données de labellisation:", error);
  },
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
  height: 50vh;
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
  color: #fff;
  background-color: rgba(0, 0, 0, 0.5);
  padding: 2px 4px;
  border-radius: 4px;
}
</style>
