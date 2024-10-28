<template>
  <div class="app-container bg-gray-900 min-h-screen flex items-center justify-center p-4">
    <!-- Composant principal de la timeline -->
    <TimelineEditor />
  </div>
</template>

<script>
import TimelineEditor from './components/TimelineEditor.vue';

export default {
  name: 'App',
  components: {
    TimelineEditor,
  },
};
</script>

<style>
/* Styles globaux pour s'assurer que l'application prend tout l'écran */
.app-container {
  font-family: Arial, sans-serif;
  color: #ffffff;
}
</style>
