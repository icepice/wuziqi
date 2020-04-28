<!-- @/components/DomVerson.vue -->
<template>
  <div class="dom_verson" :style="domVersonStyle">
    <div 
      class="horizontal_line" 
      :style="item.style"
      v-for="item in horizontalList" :key="item.id">
    </div>
    <div 
      class="vertical_line" 
      :style="item.style"
      v-for="item in verticalList" :key="item.id">
    </div>
    <!-- 棋子集合 -->
    <div class="chessmanList">
      <div 
        class="chessman" 
        :style="item.style" 
        v-for="item in chessmanList" :key="item.id">
      </div>
    </div>
    <!-- mask用于遮罩，监听鼠标的点击 -->
    <div 
      class="mask" 
      :style="maskStyle" 
      @click="!gameOver && listenDropChessman($event)">
    </div>
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
      domVersonStyle: null,
      maskStyle: null,
      horizontalList: [],  // [{id:String, style:Style}, ...]
      verticalList: [],
      chessmanList: []  // [{id:String, style:Style}, ...]
    };
  },

  methods: {
    /**
     * 绘制棋盘
     */
    drawChessboard() {
      const { count, padding, color } = this.chessboardStyle;
      this.domVersonStyle = {
        width: (count + 1) * padding + 'px',
        height: (count + 1) * padding + 'px',
      };
      // 遮罩层的宽高等于整个棋盘宽高
      this.maskStyle = {
        width: (count + 1) * padding + 'px',
        height: (count + 1) * padding + 'px',
        top: 0,
        left: 0
      };
      for(let i = 0; i < count; i++) {
        // 画横线
        this.horizontalList.push({
          id: `hor_${i}`,
          style: {
            width: (count - 1) * padding + 'px',
            top: (i + 1) * padding + 'px',
            left: padding + 'px',
            borderTop: `1px ${color} solid`
          }
        });
        // 画竖线
        this.verticalList.push({
          id: `ver_${i}`,
          style: {
            height: (count - 1) * padding + 'px',
            top: padding + 'px',
            left: (i + 1) * padding + 'px',
            borderLeft: `1px ${color} solid`
          }
        })
      }
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
      console.log(this)
      const { padding } = this.chessboardStyle;
      const { radius, blackColor, whiteColor } = this.chessmanStyle;
      this.chessmanList.push({
        id: `${x}_${y}`,
        style: {
          width: radius * 2 + 'px',
          height: radius * 2 + 'px',
          left: (x + 1) * padding - radius + 'px',
          top: (y + 1) * padding - radius + 'px',
          backgroundColor: color === 'blackColor' ? blackColor : whiteColor,
        }
      });
      return {x, y};
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
      this.chessmanList = [];
    },

    /**
     * @method public 悔棋
     * @param {PreStep} preStep {x:number  // 矩阵x坐标, y:number  // 矩阵y坐标}
     */
    unDo(preStep) {
      this.chessmanList.pop();
    }
  },

  created() {
    this.drawChessboard()
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
