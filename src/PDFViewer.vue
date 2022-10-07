<template>
  <div class="drag-box" id="dragBox" @scroll="scrollfun($event)">
    <el-scrollbar style="height: 100%; overflow-y: hidden">
      <div class="wrapper" id="pdf-container">
        <div
          v-for="item in totals"
          :id="`page-${item}`"
          :key="item"
          class="pdf-box"
        >
          <canvas :id="'canvas-pdf-' + item" class="canvas-pdf"></canvas>
        </div>
      </div>
    </el-scrollbar>
  </div>
</template>

<script>
import * as pdfjs from "pdfjs-dist";

import { TextLayerBuilder } from "pdfjs-dist/web/pdf_viewer";
import "pdfjs-dist/web/pdf_viewer.css";

import pdfjsWorker from "pdfjs-dist/build/pdf.worker.entry";
pdfjs.GlobalWorkerOptions.workerSrc = pdfjsWorker;

import { ElLoading } from "element-plus";
import { ElMessage } from "element-plus";

export default {
  name: "showPdf",
  data() {
    return {
      scale: 2.15,
      totals: [],
      pageNo: 1,
      viewHeight: 0,
      pdfUrl: null,
    };
  },
  mounted() {
    const path = location.search.replace("?url=", "");
    console.log(path);
    if (!path || path === "") {
      ElMessage({
        message: "无效地址",
        type: "error",
      });
      return;
    }
    this.pdfUrl = path;
    this.renderPdf(this.scale);
  },
  watch: {
    scale(val) {
      this.totals = [];
      this.renderPdf(val);
    },
  },
  methods: {
    async renderPdf(scale) {
      const loading = ElLoading.service({
        lock: true,
        text: "加载中...",
        background: "rgba(0, 0, 0, 0.7)",
      });

      const pdf = await pdfjs.getDocument(this.pdfUrl).promise;
      // 得到PDF的总的页数
      let totalPage = pdf.numPages;
      let idName = "canvas-pdf-";
      // 根据总的页数创建相同数量的canvas
      this.createCanvas(totalPage, idName);

      for (let i = 1; i <= totalPage; i++) {
        const page = await pdf.getPage(i);
        let pageDiv = document.getElementById(`page-${i}`);
        let viewport = page.getViewport({ scale: scale });

        let canvas = document.getElementById(idName + i);
        let context = canvas.getContext("2d");
        canvas.height = viewport.height;
        canvas.width = viewport.width;
        this.viewHeight = viewport.height;
        let renderContext = {
          canvasContext: context,
          viewport,
        };
        // 如果你只是展示pdf而不需要复制pdf内容功能，则可以这样写render
        // page.render(renderContext) 如果你需要复制则像下面那样写利用text-layer
        await page.render(renderContext).promise;
        const textContent = await page.getTextContent();
        // 创建文本图层div
        const textLayerDiv = document.createElement("div");
        textLayerDiv.setAttribute("class", "textLayer");
        // 将文本图层div添加至每页pdf的div中
        pageDiv.appendChild(textLayerDiv);
        // 创建新的TextLayerBuilder实例
        let textLayer = new TextLayerBuilder({
          textLayerDiv: textLayerDiv,
          pageIndex: page.pageIndex,
          viewport: viewport,
          eventBus: {
            dispatch: (evtName, res) => {},
          },
        });
        textLayer.setTextContent(textContent);
        textLayer.render();
        loading.close();
      }
    },
    createCanvas(totalPages) {
      for (let i = 1; i <= totalPages; i++) {
        this.totals.push(i);
      }
    },
    // 分页
    scrollfun(e) {
      let scrollTop = e.target.scrollTop;
      if (scrollTop === 0) {
        this.pageNo = 1;
      } else {
        this.pageNo = Math.ceil(scrollTop / this.viewHeight);
      }
    },
    // 放大
    scalBig() {
      this.scale = this.scale + 0.1;
    },
    // 缩小
    scalSmall() {
      if (this.scale > 1.2) {
        this.scale = this.scale - 0.1;
      }
    },
  },
};
</script>

<style scoped lang="less">
.drag-box {
  height: 100%;
}
.pdf-box {
  position: relative;
}
.el-scrollbar__wrap {
  overflow-x: hidden;
}
</style>

