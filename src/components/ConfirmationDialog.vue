<!-- src/components/ConfirmationDialog.vue -->
<template>
  <div class="dialog-overlay" v-if="visible">
    <div class="dialog-box">
      <p>{{ message }}</p>
      <div class="dialog-actions">
        <button @click="$emit('confirm')">Oui</button>
        <button @click="$emit('cancel')">Non</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    visible: {
      type: Boolean,
      default: false,
    },
    message: {
      type: String,
      default: "Êtes-vous sûr de vouloir supprimer cet élément?",
    },
  },
};
</script>

<style scoped>
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.dialog-box {
  background: #333;
  padding: 20px;
  border-radius: 8px;
  color: white;
  text-align: center;
}

.dialog-actions {
  margin-top: 15px;
  display: flex;
  justify-content: space-around;
}

.dialog-actions button {
  background-color: #555;
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.dialog-actions button:hover {
  background-color: #777;
}
</style>
