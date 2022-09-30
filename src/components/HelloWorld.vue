<template>
  <div class="pdf-render">
    <div style="text-align:center">
      <span>
        当前pdf总页数为: {{totalPageNum}}
      </span>
    </div>
    <div :style="`margin:0 auto;width:${pdfWidth};`">
      <canvas
        v-for="page in totalPageNum"
        :key="page" 
        :id="'pdfCanvas' + page"></canvas>
    </div>
  </div>
</template>

<script>
const PDFJS = require("pdfjs-dist")
PDFJS.GlobalWorkerOptions.workerSrc = "https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.5.207/pdf.worker.js"
export default {
  name: 'pdf-render',
  data () {
    return {
      pdfSrc: "",
      pdfDoc: null, //文档内容
      pdfWidth: "",
      // pageNum: 1,
      pageRendering: false,
      pageNumPending: null, //待处理的页码
      totalPageNum: [],
      scale: 1  //放大倍数
    };
  },
  mounted() {
    this.getPdfUrl();
  },
  methods: {
    //获取本地pdf的url
    getPdfUrl(){
      this.pdfSrc = "/static/2022创造营学员手册.pdf"
      this.loadFile(this.pdfSrc)
    },
    loadFile(url){
      let loadingTask = PDFJS.getDocument(url);
      loadingTask.promise.then(pdf => {
        this.pdfDoc = pdf;
        this.totalPageNum = pdf.numPages;
        //$nextTick()作用：
        this.$nextTick(()=>{
          this.renderPage(1); //初始渲染首页
        })
      })
    },
    renderPage(num) {
      const that = this;
      this.pdfDoc.getPage(num).then(page => {
        let canvas = document.getElementById("pdfCanvas" + num);
        let ctx = canvas.getContext("2d");
        let dpr = window.devicePixelRatio || 1;
        let bsr =
          ctx.webkitBackingStorePixelRatio ||
          ctx.mozBackingStorePixelRatio ||
          ctx.msBackingStorePixelRatio ||
          ctx.oBackingStorePixelRatio ||
          ctx.backingStorePixelRatio ||
          1;
        let ratio = dpr / bsr;
        let viewport = page.getViewport({ scale: this.scale });
        canvas.width = viewport.width * ratio;
        canvas.height = viewport.height * ratio;

        canvas.style.width = viewport.width + "px";

        that.pdfWidth = viewport.width + "px";

        canvas.style.height = viewport.height + "px";

        ctx.setTransform(ratio, 0, 0, ratio, 0, 0);


        //将pdf页面渲染到canvas上下文中
        let renderContext = {
            canvasContext: ctx,
            viewport: viewport
        };

        page.render(renderContext);
        if(this.totalPageNum>num){
          this.renderPage(num + 1);
        }
        
      })
    }
  },
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


