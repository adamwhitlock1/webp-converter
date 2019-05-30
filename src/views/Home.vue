<template>
  <div class="home" id="home-route">
    <DragAndDrop />
  </div>
</template>

<script>
// @ is an alias to /src
import DragAndDrop from "@/components/DragAndDrop.vue";

export default {
  name: "home",
  components: {
    DragAndDrop
  }
};
</script>
