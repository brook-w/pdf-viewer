<template>
  <a
    class="github"
    target="_blank"
    href="https://github.com/brook-w/pdf-viewer"
    title="源码实现"
  ></a>
  <PDFViewer></PDFViewer>
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
.github {
  position: fixed;
  right: 50px;
  top: 40px;
  z-index: 100;
  background-image: url("./assets/img/github.png");
  background-repeat: no-repeat;
  background-size: cover;
  width: 48px;
  height: 48px;
}

</style>