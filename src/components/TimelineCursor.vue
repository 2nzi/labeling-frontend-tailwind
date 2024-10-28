<template>
    <div
      class="timeline-cursor"
      :style="{ left: `${((currentTime - scrollOffset) / (duration / zoomLevel)) * 100}%` }"
    ></div>
  </template>
  
  <script>
  export default {
    props: ["currentTime", "duration", "zoomLevel", "scrollOffset"],
  };
  </script>
  
  <style scoped>
  .timeline-cursor {
    position: absolute;
    top: 0;
    height: 100%;
    width: 2px;
    background-color: #00f;
  }
  </style>
  