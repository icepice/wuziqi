<template>
  <div class="chessboard">
    <div class="title">-- 五子棋 {{ verson }} --</div>
    <div class="tips">{{ tips }}</div>
    <div class="content">
      <!-- canver verson start -->
      <div v-show="isCanvas">
        <canvas-verson
          ref="canvasVer"
          :chessboardStyle="chessboardStyle"
          :chessmanStyle="chessmanStyle"
          :gameOver="gameOver"
          @listenDropChessman="listenDropChessman">          
        </canvas-verson>
      </div>
      <!-- canver verson end -->
      
      <!-- dom verson start -->
      <div v-show="!isCanvas">
        <dom-verson   
          ref="domVer"     
          :chessboardStyle="chessboardStyle"   
          :chessmanStyle="chessmanStyle"   
          :gameOver="gameOver"
          @listenDropChessman="listenDropChessman">
        </dom-verson>
      </div>
      <!-- dom verson end -->
      <div class="aside">
        <div @click="reStart">重新开始</div>
        <div @click="unDo" disabled>悔棋</div>
        <div @click="cancelUndo">撤销悔棋</div>
        <div @click="toggleVer">切换版本</div>
      </div>
    </div>
  </div>
</template>

<script>
import DomVerson from "@/components/DomVerson.vue";
import CanvasVerson from "@/components/CanvasVerson.vue";
export default {
  name: "Chessboard",
  components: {
    DomVerson,
    CanvasVerson
  },
  data() {
    return {
      chessboardStyle: {
        count: 16, // 交叉点数
        padding: 40, // 边距
        color: "#333" // 边框颜色
      },
      chessboardMatrix: [], // 棋盘矩阵；0：没有棋子，1：黑棋，2：白棋
      isBlack: true, // 是否为黑棋
      tips: "黑方的回合",
      preStep: null,  // {x:number  // 矩阵x坐标, y:number  // 矩阵y坐标}
      canUndo: false, // 是否允许悔棋
      gameOver: false, // 是否游戏结束；true：游戏结束
      isCanvas: false // 当前是否是canvas
    };
  },

  watch: {
    isBlack(val) {
      this.tips = val === true ? "黑方的回合" : "白方的回合";
    },
  },

  computed: {
    // 棋盘的版本
    verson() {
      return this.isCanvas ? 'Canvas版' : 'Dom版';
    },

    // 棋子样式
    chessmanStyle() {
      return {
        radius: this.chessboardStyle.padding * 0.4,
        blackColor: '#000',
        whiteColor: '#E3E3E3'
      }
    }
  },

  methods: {
    /**
     * 初始化棋盘矩阵，用于记录棋盘上棋子的信息
     */
    initChessboardMatrix() {
      const { count } = this.chessboardStyle;
      const chessboardMatrix = this.chessboardMatrix;

      for (let x = 0; x < count; x++) {
        chessboardMatrix[x] = [];
        for (let y = 0; y < count; y++) {
          chessboardMatrix[x][y] = 0;
        }
      }
    },

    /**
     * 监听下棋
     * @param {Event} e 事件对象
     * @param {Function} cb_drawChessman 绘制棋子的回调
     */
    listenDropChessman(e, cb_drawChessman) {
      const { isBlack } = this;
      const { padding, count } = this.chessboardStyle;

      let { offsetX: x, offsetY: y } = e;
      x = Math.round(x / padding) - 1; // 代表矩阵的行
      y = Math.round(y / padding) - 1; // 代表矩阵的列
      if (x + 1 === 0 || x === count || y + 1 === 0 || y === count) return;  // 棋盘边缘不能下棋
      if (this.chessboardMatrix[x][y] != 0) return;  // 已有棋子的位置不能下棋
      
      this.preStep = cb_drawChessman(x, y, isBlack ? 'blackColor' : 'whiteColor');
      this.chessboardMatrix[x][y] = isBlack ? 1 : 2; // 更新矩阵

      if (this.isWin(x, y) > 0) {
        this.tips = isBlack
          ? "！！！ 黑方获胜 ！！！"
          : "！！！ 白方获胜 ！！！";
        this.gameOver = true;
        return;
      }
      this.canUndo = true; // 落子后才可以悔棋
      this.isBlack = !isBlack; // 切换角色
    },

    /**
     * 判断是否产生获胜者
     * @param {Number} x 落子所在的矩阵的行
     * @param {Number} y 落子所在的矩阵的列
     * @return {Number} 总共连杀数
     */
    isWin(x, y) {
      const { chessboardMatrix } = this;
      const { count } = this.chessboardStyle;
      const arrX = chessboardMatrix.map(item => item[y]); // x轴上连杀
      const arrY = chessboardMatrix[x]; // y轴上连杀
      const arrSlash1 = []; // 左上至右下轴上连杀
      const arrSlash2 = []; // 右上至左下轴上连杀
      let totalScore = 0; // 总共连杀分数

      chessboardMatrix.forEach((_y, i) => {
        // 左斜线
        const S1Item = _y[y - (x - i)];
        if (S1Item !== undefined) {
          arrSlash1.push(S1Item);
        }
        // 右斜线
        const S2Item = _y[y + (x - i)];
        if (S2Item !== undefined) {
          arrSlash2.push(S2Item);
        }
      });

      [arrX, arrY, arrSlash1, arrSlash2].forEach(item => {
        let temp = item.some((v, k) => {
          if (
            v !== 0 &&
            item[k + 1] === v &&
            item[k + 2] === v &&
            item[k + 3] === v &&
            item[k + 4] === v
          ) {
            return true;
          }
        });
        if (temp) {
          totalScore++;
        }
      });
      return totalScore;
    },

    /**
     * 从新开始游戏
     */
    reStart() {
      const currentVer = this.isCanvas ? 'canvasVer' : 'domVer';

      this.$refs[currentVer].reset();
      this.initChessboardMatrix();
      this.isBlack = true;
      this.preStep = null;
      this.gameOver = false;
      this.tips = "黑方的回合";
    },

    /**
     * 悔棋
     */
    unDo() {
      if (!this.preStep) return;  // 没有落子
      const { preStep, chessboardMatrix, canUndo, gameOver } = this;
      const { count, padding, color } = this.chessboardStyle;
      const { x, y } = this.preStep;
      const currentVer = this.isCanvas ? 'canvasVer' : 'domVer';
      if (gameOver) return;  // 游戏结束
      if (!canUndo) return;  // 不允许连续悔棋

      chessboardMatrix[x][y] = 0; // 重置矩阵到上一步
      this.isBlack = !this.isBlack; // 返回上一角色
      this.canUndo = false; // 不允许连续悔棋
      this.$refs[currentVer].unDo(preStep);
    },

    /**
     * 撤销悔棋
     */
    cancelUndo() {
      if (!this.preStep) return; // 没有落子
      const { chessboardMatrix, isBlack, gameOver, canUndo, preStep } = this;
      const { x, y } = this.preStep;
      const currentVer = this.isCanvas ? 'canvasVer' : 'domVer';
      
      if (gameOver) return; // 游戏结束
      if (canUndo) return;  // 悔棋后才能撤销悔棋
      this.canUndo = true; // 撤销悔棋后又可以悔棋
      this.preStep = this.$refs[currentVer].drawChessman(x, y, isBlack ? 'blackColor' : 'whiteColor');
      this.isBlack = !this.isBlack;
      chessboardMatrix[x][y] = isBlack ? 1 : 2;
    },

    /**
     * 在canvas和dom之间进行切换
     */
    toggleVer() {
      this.reStart(); // 重置棋盘
      this.isCanvas = !this.isCanvas;
    }
  },

  created() {
    this.initChessboardMatrix();
  },
};
</script>

<style scope>
.chessboard {
  --main-color: #666;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  user-select: none;
  font-size: 24px;
  font-weight: bold;
  color: var(--main-color);
}

.title {
  padding-bottom: 30px;
}

.tips {
  padding-bottom: 30px;
  color: #992222;
}

.content {
  display: flex;
}

.aside {
  margin: 10px 60px;
}

.aside div {
  flex: 1;
  border: 1px var(--main-color) solid;
  border-radius: 4px;
  margin-bottom: 50px;
  padding: 8px;
  width: 34px;
  box-shadow: 1px 1px 2px 1px #eee;
}

.aside div:active {
  opacity: 0.6;
}

.aside div:hover {
  background-color: #efefef;
}
</style>
