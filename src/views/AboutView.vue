<template>
  <div>
    <div id="canvasDiv"></div>
    <button id="btn_submit" @click="submit">提交</button>
    <button id="btn_submit" @click="clear">清空</button>
  </div>
</template>
<script>
export default {
  data() {
    return {
      canvasDiv: null,
      offsetTop: 0,
      offsetLeft: 0,
      mouseX: 0,
      mouseY: 0,
      clickX: [],
      clickY: [],
      clickDrag: [],
      paint: false,
      point: {}
    };
  },
  created() {},
  mounted() {
    this.initCanvas();
  },
  methods: {
    initCanvas() {
      //初始化Canvas
      this.canvasDiv = document.getElementById("canvasDiv");
      //获取当前元素距离屏幕边缘的距离
      this.offsetLeft = this.canvasDiv.getBoundingClientRect().left;
      this.offsetTop = this.canvasDiv.getBoundingClientRect().top;
      //生产Canva画板
      this.canvas = document.createElement("canvas");
      let canvasWidth = this.canvasDiv.clientWidth;
      this.canvas.setAttribute("width", canvasWidth);
      this.canvas.setAttribute("height", 400);
      this.canvasDiv.appendChild(this.canvas);
      this.context = this.canvas.getContext("2d");
      this.point.notFirst = false;
      //添加监听事件
      this.canvas.addEventListener("mousedown", this.mouseDown);
      this.canvas.addEventListener("mousemove", this.mouseMove);
      this.canvas.addEventListener("mouseup", this.mouseUp);
      this.canvas.addEventListener("mouseleave", this.mouseUp);
      this.canvas.addEventListener("touchstart", this.touchStart);
      this.canvas.addEventListener("touchmove",this.touchMove);
      this.canvas.addEventListener("touchend",this.touchEnd)
      window.addEventListener("resize", this.resizeOffset);
    },
    //鼠标点击事件
    mouseDown(e) {
      this.mouseX = e.pageX - this.offsetLeft;
      this.mouseY = e.pageY - this.offsetTop;
      this.paint = true;
      this.addClick(this.mouseX, this.mouseY);
      this.redraw();
    },
    mouseMove(e) {
      if (this.paint) {
        let x = e.pageX - this.offsetLeft;
        let y = e.pageY - this.offsetTop;
        this.addClick(x, y, true);
        this.redraw();
      }
    },
    mouseUp(e) {
      this.paint = false;
    },
    touchStart(e){
      e.preventDefault();
      this.mouseX = e.touches[0].pageX - this.offsetLeft;
      this.mouseY = e.touches[0].pageY - this.offsetTop;
      this.paint = true;
      this.addClick(this.mouseX, this.mouseY);
      this.redraw();
    },
    touchMove(e){
      e.preventDefault();
      if (this.paint) {
        let x = e.touches[0].pageX - this.offsetLeft;
        let y = e.touches[0].pageY - this.offsetTop;
        this.addClick(x, y, true);
        this.redraw();
      }
    },
    touchEnd(){
      this.paint = false;
    },
    addClick(x, y, dragging) {
      this.clickX.push(x);
      this.clickY.push(y);
      this.clickDrag.push(dragging);
    },
    //绘制Canvas
    redraw() {
      this.context.strokeStyle = "#000";
      this.context.lineJoin = "round";
      this.context.lineWidth = 2;
      while (this.clickX.length > 0) {
        this.point.bx = this.point.x;
        this.point.by = this.point.y;
        this.point.x = this.clickX.pop();
        this.point.y = this.clickY.pop();
        this.point.drag = this.clickDrag.pop();
        this.context.beginPath();
        if (this.point.drag && this.point.notFirst) {
          this.context.moveTo(this.point.bx, this.point.by);
        } else {
          this.point.notFirst = true;
          this.context.moveTo(this.point.x - 1, this.point.y);
        }
        this.context.lineTo(this.point.x, this.point.y);
        this.context.closePath();
        this.context.stroke();
      }
    },
    submit() {
      var blank = document.createElement('canvas');//系统获取一个空canvas对象
      blank.width = this.canvas.width;
      blank.height = this.canvas.height;
      let isNull=this.canvas.toDataURL()==blank.toDataURL();
      console.log(isNull);
      console.log(this.canvas.toDataURL("image/png"));
    },
    clear(){
      this.canvas.width=this.canvas.width;
    },
    resizeOffset() {
      this.offsetLeft = this.canvasDiv.getBoundingClientRect().left;
      this.offsetTop = this.canvasDiv.getBoundingClientRect().top;
    }
  }
};
</script>
<style lang="less" scoped>
#canvasDiv {
  border: 1px solid #000;
}
</style>
