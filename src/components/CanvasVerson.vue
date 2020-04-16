<!-- @/components/CanvasVerson.vue -->
<template>
  <div class="canvas_wrap" :style="wrapStyle">
    <canvas 
      class="canvas" 
      v-show="!loading" 
      ref="linesCanvas">
      <p>您的浏览器版本过低，请升级您的浏览器</p>
    </canvas>
    <canvas 
      class="canvas" 
      v-show="!loading" 
      ref="chessmanCanvas" 
      @click="!gameOver && listenDropChessman($event)">
    </canvas>
  </div>
</template>

<script>
export default {
  name: 'CanvasVerson',
  props: {
    // 棋盘样式
    chessboardStyle: {
      type: Object,
      required: true
    },

    // 棋子样式
    chessmanStyle: {
      type: Object,
      required: true
    },

    // 游戏是否结束
    gameOver: {
      type: Boolean,
      required: true
    },
  },

  data () {
    return {
      wrapStyle: null,
      linesCanvas: null,
      linesCtx: null,
      chessmanCanvas: null,
      chessmanCtx: null,
      loading: true
    };
  },

  methods: {
    /**
     * 初始化棋盘容器大小
     */
    initCanWrpa() {
      const { count, padding } = this.chessboardStyle;
      this.wrapStyle = {
        width: padding * (count + 1) + 'px',
        height: padding * (count + 1) + 'px'
      }
    },

    /**
     * 初始化canvas
     */
    initCanvas() {
      const { count, padding } = this.chessboardStyle;

      this.linesCanvas = this.$refs.linesCanvas;
      this.linesCtx = this.linesCanvas.getContext("2d");
      this.chessmanCanvas = this.$refs.chessmanCanvas;
      this.chessmanCtx = this.chessmanCanvas.getContext("2d");
      this.linesCanvas.width = this.linesCanvas.height = this.chessmanCanvas.width = this.chessmanCanvas.height = padding * (count + 1);
    },

    /**
     * 定义线条路径，在原来的位置加0.5，才能画出1px，否则是2px和出现颜色方面的问题
     * @param {Context} ctx 绘图上下文
     * @param {Point} start 起点 Point {x:number, y:number}
     * @param {Point} end 终点 Point {x:number, y:number}
     */
    drawLine(ctx, start, end) {
      ctx.moveTo(start.x + 0.5, start.y + 0.5);
      ctx.lineTo(end.x + 0.5, end.y + 0.5);
    },

    /**
     * 绘制棋盘
     */
    drawChessboard() {
      const { count, padding, color } = this.chessboardStyle;
      const { linesCtx: ctx } = this;

      ctx.save(); // 将样式属性入栈
      ctx.strokeStyle = color;
      ctx.beginPath(); // 清空路径
      for (var i = 0; i < count; i++) {
        const lineOffset = i + 1; // 线条偏移
        // 画竖线
        this.drawLine(
          ctx,
          { x: lineOffset * padding, y: padding },
          { x: lineOffset * padding, y: count * padding }
        );
        // 画横线
        this.drawLine(
          ctx,
          { x: padding, y: lineOffset * padding },
          { x: count * padding, y: lineOffset * padding }
        );
      }
      ctx.stroke();
      ctx.restore(); // 将样式属性出栈
    },

    /**
     * @method public 绘制棋子，公开调用的形式：在点击撤销悔棋时调用
     *    以回调的形式，在落子时调用
     * @param {Number} x 棋子的矩阵x坐标
     * @param {Number} y 棋子的矩阵y坐标
     * @param {String} color 棋子颜色 'blackColor' 或者 'whiteColor'
     * @return {PreStep} {x:number  // 矩阵x坐标, y:number  // 矩阵y坐标}
     */
    drawChessman(x, y, color) {
      const { chessmanCtx: ctx } = this;
      const { padding } = this.chessboardStyle;
      const { radius, whiteColor, blackColor } = this.chessmanStyle;

      ctx.save();
      ctx.fillStyle = color === 'blackColor' ? blackColor : whiteColor;
      ctx.beginPath();
      ctx.arc((x + 1) * padding + 0.5, (y + 1) * padding + 0.5, radius, 0, 2 * Math.PI);
      ctx.stroke();
      ctx.fill();
      ctx.restore();
      return {x, y};
    },

    /**
     * 监听下棋
     * @param {Event} e 事件对象
     */
    listenDropChessman(e) {
      this.$emit('listenDropChessman', e, this.drawChessman)
    },

    /**
     * @method public 重置棋盘，点击切换版本、重新开始时调用
     */
    reset() {
      const { chessmanCanvas } = this;
      chessmanCanvas.width = chessmanCanvas.width;
    },

    /**
     * @method public 悔棋
     * @param {PreStep} preStep {x:number  // 矩阵x坐标, y:number  // 矩阵y坐标}
     */
    unDo(preStep) {
      const { count, padding, color } = this.chessboardStyle;
      const { chessmanCtx: ctx } = this;
      const { x, y } = preStep;
      const _x = (x + 1) * padding; // 上一步棋子所在的交叉点x
      const _y = (y + 1) * padding; // 上一步棋子所在的交叉点y

      ctx.clearRect(
        (x + 0.5) * padding + 0.5,
        (y + 0.5) * padding + 0.5,
        padding,
        padding
      );
    }
  },

  created() {
    this.initCanWrpa();
  },

  mounted() {
    this.initCanvas();
    this.drawChessboard();
    this.loading = false;
  }
}
</script>
<style scoped>
.canvas_wrap {
  position: relative;
  box-shadow: 2px 2px 4px 8px #eee;
}
.canvas {
  position: absolute;
  top: 0;
  left: 0;
}
</style>
