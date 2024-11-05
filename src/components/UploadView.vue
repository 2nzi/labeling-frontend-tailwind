<template>
    <div class="upload-container">
      <h2>Upload Your Video</h2>
      <label class="drop-zone" @dragover.prevent @drop.prevent="handleDrop">
        <input type="file" @change="handleVideoUpload" accept="video/*" class="file-input" />
        <div class="drop-zone-content">
          <p>Drag & drop your video here or click to select</p>
        </div>
      </label>
      <div v-if="loading" class="loading-section">
        <p>Uploading... {{ progress }}%</p>
        <div class="loader"></div>
      </div>
      <p v-if="errorMessage" class="error">{{ errorMessage }}</p>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        loading: false,
        progress: 0,
        errorMessage: "",
      };
    },
    methods: {
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
            this.$emit("videoUploaded", data.compressed_video_path);
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
    },
  };
  </script>
  
  <style scoped>
  .upload-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;
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
  
  .error {
    color: red;
    margin-top: 10px;
  }
  </style>
  