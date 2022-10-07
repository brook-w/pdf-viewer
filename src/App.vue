<template>
  <PDFViewer></PDFViewer>
  <div></div>
</template>
 
<script lang='ts'>
import { defineComponent, onMounted, ref } from "vue";
import PDFViewer from "./PDFViewer.vue";
export default defineComponent({
  components: {
    PDFViewer,
  },
  setup() {},
});
</script>
 
<style scoped>
</style>