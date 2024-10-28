<template>
  <div class="h-10 border-t border-gray-600 flex items-center pl-4 text-white">
    {{ event.name }}
  </div>
</template>

<script>
export default {
  name: "TimelineEventRow",
  props: ["event"],
};
</script>
