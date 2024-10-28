<template>
    <div class="confirm-dialog-overlay">
      <div class="confirm-dialog">
        <h3>Confirmer la suppression</h3>
        <div class="confirm-dialog-buttons">
          <button class="confirm-btn" @click="$emit('confirm')">Oui</button>
          <button class="cancel-btn" @click="$emit('cancel')">Non</button>
        </div>
      </div>
    </div>
  </template>
  
  <style scoped>
  /* Styles pour la boîte de dialogue de confirmation */
  .confirm-dialog-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .confirm-dialog {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
  }
  </style>
  