<template>
    <button @click="handleSave" :disabled="loading" class="save-button">
      <span v-if="loading">Saving...</span>
      <span v-else>Save Labeling</span>
    </button>
  </template>
  
  <script>
  export default {
    props: {
      generateLabelingData: {
        type: Function,
        required: true
      },
      saveUrl: {
        type: String,
        default: "http://localhost:8000/api/save_labeling" // Default URL, can be customized
      }
    },
    data() {
      return {
        loading: false
      };
    },
    methods: {
      async handleSave() {
        this.loading = true;
        const labelingData = this.generateLabelingData();
  
        try {
          const response = await fetch(this.saveUrl, {
            method: "POST",
            headers: {
              "Content-Type": "application/json"
            },
            body: JSON.stringify(labelingData)
          });
  
          if (response.ok) {
            this.$emit("save-success");
            console.log("Labeling data saved successfully.");
          } else {
            console.error("Failed to save labeling data:", response.status);
            this.$emit("save-error", response.status);
          }
        } catch (error) {
          console.error("Error saving labeling data:", error);
          this.$emit("save-error", error);
        } finally {
          this.loading = false;
        }
      }
    }
  };
  </script>
  
  <style scoped>
  .save-button {
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


.save-button:hover {
  background: linear-gradient(135deg, #007acc, #005ea0);
  transform: translateY(-2px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.3);
}

.save-button:disabled {
  background: #d3d3d3;
  color: #a0a0a0;
  box-shadow: none;
  cursor: not-allowed;
}
  </style>
  