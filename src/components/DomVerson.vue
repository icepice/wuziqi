<!-- @/components/DomVerson.vue -->
<template>
  <div class="dom_verson" ref="dom_verson">
    <!-- 横线模板 -->
    <div class="horizontal_line" style="display: none" ref="temp_horizontal_line"></div>

    <!-- 竖线模板 -->
    <div class="vertical_line" style="display: none" ref="temp_vertical_line"></div>

    <!-- 线条集合 -->
    <div ref="lineList"></div>

    <!-- 棋子模板 -->
    <div class="chessman" style="display: none" ref="temp_chessman"></div>

    <!-- 棋子集合 -->
    <div ref="chessmanList"></div>

    <!-- mask用于遮罩，监听鼠标的点击 -->
    <div class="mask" ref="mask" @click="!gameOver && listenDropChessman($event)"></div>
  </div>
</template>

<script>
export default {
  name: 'DomVerson',
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
    };
  },

  methods: {
    /**
     * 绘制棋盘
     */
    drawChessboard() {     
      const { count, padding, color } = this.chessboardStyle;
      const dom_verson = this.$refs.dom_verson;
      const mask = this.$refs.mask;
      const temp_horizontal_line = this.$refs.temp_horizontal_line;
      const temp_vertical_line = this.$refs.temp_vertical_line;
      // 遮罩层的宽高等于整个棋盘宽高
      mask.style.width = (count + 1) * padding + 'px';
      mask.style.height = (count + 1) * padding + 'px';
      mask.style.top = '0px';
      mask.style.left = '0px';
        
      dom_verson.style.padding = padding + 'px';
      dom_verson.style.width = (count - 1) * padding + 'px';
      dom_verson.style.height = (count - 1) * padding + 'px';

      // 画横线
      for(let i = 0; i < count; i++) {
        const horizontal_line = temp_horizontal_line.cloneNode();
        horizontal_line.style.width = (count - 1) * padding + 'px';
        horizontal_line.style.top = (i + 1) * padding + 'px';
        horizontal_line.style.left = padding + 'px';
        horizontal_line.style.borderTop = `1px ${color} solid`;
        horizontal_line.style.display = 'inline-block';
        this.$refs.lineList.appendChild(horizontal_line);
      }
      // 画竖线
      for(let i = 0; i < count; i++) {
        const vertical_line = temp_vertical_line.cloneNode();
        vertical_line.style.height = (count - 1) * padding + 'px';        
        vertical_line.style.top = padding + 'px';
        vertical_line.style.left = (i + 1) * padding + 'px';
        vertical_line.style.borderLeft = `1px ${color} solid`;
        vertical_line.style.display = 'inline-block';
        this.$refs.lineList.appendChild(vertical_line);
      }
    },

    /**
     * @method public 绘制棋子，公开调用的形式：在点击撤销悔棋时调用
     *    以回调的形式，在落子时调用
     * @param {Number} x 棋子的矩阵x坐标
     * @param {Number} y 棋子的矩阵y坐标
     * @param {String} color 棋子颜色 'blackColor' 或者 'whiteColor'
     * @return {PreStep} {x:number  // 矩阵x坐标, y:number  // 矩阵y坐标, dom?:element  // dom版才有}
     */
    drawChessman(x, y, color) {
      const { padding } = this.chessboardStyle;
      const { radius, blackColor, whiteColor } = this.chessmanStyle;
      const chessman = this.$refs.temp_chessman.cloneNode();      
      chessman.style.width = radius * 2 + 'px';
      chessman.style.height = radius * 2 + 'px';
      chessman.style.left = (x + 1) * padding - radius + 'px';
      chessman.style.top = (y + 1) * padding - radius + 'px';
      chessman.style.backgroundColor = color === 'blackColor' ? blackColor : whiteColor;
      chessman.style.display = 'inline-block';
      this.$refs.chessmanList.appendChild(chessman);      
      return {x, y, dom: chessman};
    },

    /**
     * 监听下棋
     * @param {Event} e 事件对象
     */
    listenDropChessman(e) {
      this.$emit('listenDropChessman', e, this.drawChessman);
    },

    /**
     * @method public 重置棋盘
     */
    reset() {
      this.$refs.chessmanList.innerHTML = '';
    },

    /**
     * @method public 悔棋
     * @param {PreStep} preStep {x:number  // 矩阵x坐标, y:number  // 矩阵y坐标, dom?:element  // dom版才有}
     */
    unDo(preStep) {
      this.$refs.chessmanList.removeChild(preStep.dom); 
    }
  },

  mounted() {
    this.drawChessboard();
  },
}

</script>
<style scoped>
.dom_verson {    
  position: relative;
  box-shadow: 2px 2px 4px 8px #eee;
}

.vertical_line, .horizontal_line {
  position: absolute;  
}

.chessman {
  position: absolute;
  border-radius: 50%;
  border: 1px #000 solid;
}

.mask {
  position: absolute;
  top: 0;
  left: 0;
}
</style>
