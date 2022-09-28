<template>
  <div class="pdf-render">
    <h1>pdf预览</h1>
    <div style="text-align:center">
      <span>
        当前pdf总页数为: {{totalPageNum}}
      </span>
    </div>
    <canvas id="pdfCanvas"></canvas>
  </div>
</template>

<script>
import PDFJS from 'pdfjs-dist/legacy/build/pdf'

export default {
  name: 'pdf-render',
  //props: ['pdfUrl'],
  data () {
    return {
      pageNum: 1,
      pageRendering: false,
      pageNumPending: null,
      pdfDoc: null,
      totalPageNum: 0,
      pdfUrl: 'http://mozilla.github.io/pdf.js/web/compressed.tracemonkey-pldi-09.pdf'
    }
  },
  methods: {
    renderPage(num) {
      let vm = this;
      this.pageRendering = true;
      let canvas = document.getElementById('pdfCanvas')
      this.pdfDoc.getPage(num).then(function(page){
        var viewport = page.getViewport(vm.scale);
        canvas.height = viewport.height;
        canvas.width = viewport.width;

        var renderContext = {
            canvasContext: vm.ctx,
            viewport: viewport
        };
        var renderTask = page.render(renderContext);

        renderTask.promise.then(function() {
            vm.pageRendering = false;
            if(vm.pageNumPending !== null) {
              // New page rendering is pending
              vm.renderPage(vm.pageNumPending);
              vm.pageNumPending = null;
            }
        });

      })
    }
  },
  mounted() {
    let vm = this;
    PDFJS.getDocument(vm.pdfUrl).then(function(pdfDoc_) { //初始化pdf
        vm.pdfDoc = pdfDoc_;
        vm.totalPageNum = vm.pdfDoc.numPages;
        vm.renderPage(vm.pageNum);
    });

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style type="text/less" lang="less" rel="stylesheet/less">
.pdf-render {
  h1 {
    margin: 30px auto;
    text-align: center;
    font-family: "宋体";
    letter-spacing: 2px;
  }
}
</style>
