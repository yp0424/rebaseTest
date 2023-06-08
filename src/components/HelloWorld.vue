<template>
  <div>
    <!-- <div class="body"></div> -->
    <input type="password" v-model="value">
  </div>
</template>
<script>
export default {
  data() {
    return {
      offsetLeft:9,
      offsetTop:10,
      value:""
    };
  },
  created() {},
  mounted() {
    this.init();
  },
  watch:{
  },
  mounted(){
  },
  methods: {
    change(e){
      this.value= e.target.innerText
    },
    setVal(val){
      this.$refs.div.innerHTML=val
    },
    changeText(){
      console.log(111);
    },
    init() {
      //创建canvas
      this.canvasDiv = document.getElementById("canvasDiv");
      this.canvas = document.createElement("canvas");
      var canvasWidth = 400;
      var canvasHeight = 300;
      var _this = this;
      this.context = this.canvas.getContext("2d");

      this.canvas.addEventListener("touchstart", function(e) {
        _this.mouseX = e.pageX - _this.offsetLeft;
        _this.mouseY = e.pageY - _this.offsetTop;
        _this.paint = true;
        console.log(e.pageX,this.offsetLeft);
        _this.addClick(e.pageX - _this.offsetLeft, e.pageY - _this.offsetTop);
        _this.redraw();
      });
      this.canvas.addEventListener("touchend", function(e) {
        _this.paint = false;
      });
      this.canvas.addEventListener("touchmove", function(e) {
        if (_this.paint) {
          _this.addClick(
            e.pageX - _this.offsetLeft,
            e.pageY - _this.offsetTop,
            true
          );
          _this.redraw();
        }
      });
      this.canvas.addEventListener("mousedown", function(e) {
        _this.mouseX = e.pageX - _this.offsetLeft;
        console.log(e.pageX , _this.offsetLeft,_this.offsetTop);
        _this.mouseY = e.pageY - _this.offsetTop;
        _this.paint = true;
        _this.addClick(e.pageX - _this.offsetLeft, e.pageY - _this.offsetTop);
        _this.redraw();
      });
      this.canvas.addEventListener("mousemove", function(e) {
        if (_this.paint) {
          _this.addClick(
            e.pageX - _this.offsetLeft,
            e.pageY - _this.offsetTop,
            true
          );

          _this.redraw();
        }
      });
      this.canvas.addEventListener("mouseup", function(e) {
        _this.paint = false;
      });
      this.canvas.addEventListener("mouseleave", function(e) {
        _this.paint = false;
      });

      document.addEventListener("touchmove", this.onDocumentTouchMove, false);
      _this.point = {};
      _this.point.notFirst = false;
      this.canvas.setAttribute("width", canvasWidth);
      this.canvas.setAttribute("height", canvasHeight);
      this.canvas.setAttribute("id", "canvas");
      this.canvasDiv.appendChild(this.canvas);

      this.clickX = new Array();
      this.clickY = new Array();
      this.clickDrag = new Array();
      this.paint;
    },
    addClick(x, y, dragging) {
      console.log(x);
      this.clickX.push(x);
      this.clickY.push(y);
      this.clickDrag.push(dragging);
    },
    onDocumentTouchStart(event) {
      if (event.touches.length == 1) {
        event.preventDefault();
        // Faking double click for touch devices
        var now = new Date().getTime();
        if (now - timeOfLastTouch < 250) {
          reset();
          return;
        }
        timeOfLastTouch = now;
        mouseX = event.touches[0].pageX;
        mouseY = event.touches[0].pageY;
        isMouseDown = true;
      }
    },
    /**
     * 触摸移动
     */
    onDocumentTouchMove(event) {
      if (event.touches.length == 1) {
        event.preventDefault();
        mouseX = event.touches[0].pageX;
        mouseY = event.touches[0].pageY;
      }
    },

    /**
     * 结束触摸
     */
    onDocumentTouchEnd(event) {
      if (event.touches.length == 0) {
        event.preventDefault();
        isMouseDown = false;
      }
    },
    redraw() {
      //canvas.width = canvas.width; // Clears the canvas
      this.context.strokeStyle = "#df4b26";
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
          // debugger
          this.context.moveTo(this.point.bx, this.point.by);
        } else {
          // debugger
          this.point.notFirst = true;
          this.context.moveTo(this.point.x - 1, this.point.y);
        }
        // debugger
        this.context.lineTo(this.point.x, this.point.y);
        this.context.closePath();
        this.context.stroke();
      }
    }
  }
};
</script>
<style lang="less" scoped>
canvas{
  text-align: left !important;
}
.body{
  width: 100px;
  height: 100px;
  animation: myAnim 0.5s infinite;

}
@keyframes myAnim{
  0%{
    background: url(../assets/logo.png);
    background-size: cover;
  }
  // 50%{
  //   background: url(../assets/Wechat.jpg);
  //   background-size: cover;
  // }
  100%{
     background: url(../assets/logo.png);
    background-size: cover;
  }
}
</style>
