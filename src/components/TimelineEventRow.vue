<template>
  <div
    class="timeline-event-row"
    :class="{ 'selected-row': isSelected }"
    @click="$emit('select')"
  >
    <span>{{ event.name }}</span>
  </div>
</template>

<script>
export default {
  props: ["event", "isSelected"],
};
</script>

<style scoped>
.timeline-event-row {
  height: 50px;
  border-bottom: 1px solid #555;
  display: flex;
  align-items: center;
  padding-left: 10px;
  cursor: pointer;
}
.timeline-event-row.selected-row {
  background-color: #5a5a2b; /* Couleur de surbrillance pour la sélection */
  border: 2px solid yellow; /* Bordure jaune pour la ligne sélectionnée */
}
</style>
