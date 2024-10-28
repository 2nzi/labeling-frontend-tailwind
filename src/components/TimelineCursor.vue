<template>
    <div :style="cursorStyle" class="timeline-cursor"></div>
  </template>
  
  <script>
  export default {
    name: "TimelineCursor",
    props: {
      currentTime: {
        type: Number,
        required: true,
      },
      duration: {
        type: Number,
        required: true,
      },
    },
    computed: {
      cursorStyle() {
        const position = (this.currentTime / this.duration) * 100;
        return {
          left: `${position}%`,
          height: "100%",
          backgroundColor: "#007bff",
          width: "2px",
          position: "absolute",
          top: "0",
        };
      },
    },
  };
  </script>
  
  <style scoped>
  .timeline-cursor {
    transition: left 0.1s ease-in-out;
  }
  </style>
  