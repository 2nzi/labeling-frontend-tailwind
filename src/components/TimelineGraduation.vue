<template>
  <div class="relative h-8 w-full bg-gray-800">
    <div
      v-for="mark in graduationMarks"
      :key="mark.value"
      :style="{ left: mark.position + '%' }"
      class="absolute text-gray-300 text-xs transform -translate-x-1/2"
    >
      {{ mark.label }}
    </div>
  </div>
</template>

<script>
export default {
  name: "TimelineGraduation",
  props: ["duration"],
  computed: {
    graduationMarks() {
      const interval = 5; // In seconds
      const marks = [];
      for (let i = 0; i <= this.duration; i += interval) {
        marks.push({
          label: i,
          position: (i / this.duration) * 100,
          value: i,
        });
      }
      return marks;
    },
  },
};
</script>
