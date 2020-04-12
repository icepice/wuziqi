<template>
  <div class="chessBoard">
    <div class="title">-- 五子棋 --</div>
    <div class="tips">{{ tips }}</div>
    <div class="content">
      <canvas ref="chessBoard">
        <p>您的浏览器版本过低，请升级您的浏览器</p>
      </canvas>
      <div class="aside">
        <div @click="reStart">重新开始</div>
        <div @click="unDo" disabled>悔棋</div>
        <div @click="cancelUndo">撤销悔棋</div>
      </div>
    </div>    
  </div>
</template>

<script>
export default {
  name: "ChessBoard",
  data() {
    return {
      canvas: null,
      ctx: null,
      chessBoardStyle: {        
        count: 16,  // 交叉点数        
        padding: 40,  // 边距
        color: '#333' // 边框颜色
      },
      chessBoardMatrix: [],  // 棋盘矩阵；0：没有棋子，1：黑棋，2：白棋
      isBlack: true,  // 是否为黑棋
      tips: '黑方的回合',
      preStep: null, // {x:number, y:number, isBlack:boolean, canUndo:boolean}
      gameOver: false  // 是否游戏结束；true：游戏结束
    }
  },

  watch: {
    isBlack(val) {
      this.tips = val === true ? '黑方的回合' : '白方的回合'
    }
  },

  methods: {
    /**
     * 初始化canvas
     */
    initCanvas() {      
      this.canvas = this.$refs.chessBoard;
      this.ctx = this.canvas.getContext('2d');
      this.drawChessBoard();
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
    drawChessBoard() {      
      const { canvas, ctx } = this;
      const { count, padding, color } = this.chessBoardStyle;

      canvas.width = canvas.height = padding * (count + 1);
      ctx.save();  // 将样式属性入栈
      ctx.strokeStyle = color;            
      ctx.beginPath();  // 清空路径
      for(var i = 0; i < count; i++){
        const lineOffset = i + 1;  // 线条偏移
        // 画竖线
        this.drawLine(ctx, { x: lineOffset * padding, y: padding }, { x: lineOffset * padding, y: count * padding });
        // ctx.moveTo(lineOffset * padding + 0.5, padding + 0.5);
        // ctx.lineTo(lineOffset * padding + 0.5, count * padding + 0.5);
        // 画横线
        this.drawLine(ctx, { x: padding, y: lineOffset * padding }, { x: count * padding, y: lineOffset * padding });
        // ctx.moveTo(padding + 0.5, lineOffset * padding + 0.5);
        // ctx.lineTo(count * padding + 0.5, lineOffset * padding + 0.5);
        ctx.stroke();
      }
      ctx.restore();  // 将样式属性出栈
    },

    /** 
     * 初始化棋盘矩阵，用于记录棋盘上棋子的信息
     */
    initChessboardMatrix() {
      const { count } = this.chessBoardStyle;
      const chessBoardMatrix = this.chessBoardMatrix;

      for(let x = 0; x < count; x++){
        chessBoardMatrix[x] = [];
        for(let y = 0; y < count; y++){
            chessBoardMatrix[x][y] = 0;
        }
      }      
    },

    /**
     * 绘制棋子
     * @param {Number} x 棋子x坐标；为边距的整数倍
     * @param {Number} y 棋子y坐标；为边距的整数倍
     */
    drawChessman(x, y) {
      const { ctx } = this;
      const { padding } = this.chessBoardStyle;

      ctx.save();
      ctx.fillStyle = this.isBlack ? '#000' : '#E3E3E3';
      ctx.beginPath();
      ctx.arc(x + 0.5, y + 0.5, padding * 0.4, 0, 2 * Math.PI);
      ctx.stroke();
      ctx.fill();
      ctx.restore();
    },

    /**
     * 监听下棋
     */
    listenDropChessman() {
      const { canvas } = this;
      const { padding, count } = this.chessBoardStyle;

      canvas.onclick = (e) => {
        let { offsetX: x, offsetY: y } = e;
        x = Math.round(x / padding) - 1;  // 代表矩阵的行
        y = Math.round(y / padding) - 1;  // 代表矩阵的列
        if(x + 1 === 0 || x === count || y + 1 === 0 || y === count) return console.log('无效点击');
        if(this.chessBoardMatrix[x][y] != 0) return console.log('该位置已有棋子');

        this.drawChessman((x + 1) * padding, (y + 1) * padding);
        this.chessBoardMatrix[x][y] = this.isBlack ? 1 : 2; // 更新矩阵
                
        if(this.isWin(x, y) > 0) {
          this.tips = this.isBlack ? '！！！ 黑方获胜 ！！！' : '！！！ 白方获胜 ！！！';
          canvas.onclick = null;
          this.gameOver = true;
          // window.setTimeout(() => window.alert(this.tips), 0);                      
          return;
        };
        this.preStep = { x, y, isBlack: this.isBlack, canUndo: true }; // 记录操作        
        this.isBlack = !this.isBlack;  // 切换角色        
      }
    },

    /**
     * 判断是否产生获胜者
     * @param {Number} x 落子所在的矩阵的行
     * @param {Number} y 落子所在的矩阵的列
     * @return {Number} 总共连杀数
     */
    isWin(x, y) {
      const { chessBoardMatrix } = this;
      const { count } = this.chessBoardStyle;
      const arrX = chessBoardMatrix.map(item => item[y]);  // x轴上连杀
      const arrY = chessBoardMatrix[x];  // y轴上连杀
      const arrSlash1 = [];  // 左上至右下轴上连杀
      const arrSlash2 = [];  // 右上至左下轴上连杀
      let totalScore = 0; // 总共连杀分数

      chessBoardMatrix.forEach((_y, i) => {
        // 左斜线
        const S1Item = _y[y - (x - i)];
        if(S1Item !== undefined){
            arrSlash1.push(S1Item);
        }
        // 右斜线
        const S2Item = _y[y + (x - i)];
        if(S2Item !== undefined) {
            arrSlash2.push(S2Item);
        }
      });
      
      [arrX, arrY, arrSlash1, arrSlash2].forEach((item) => {
        let temp = item.some((v, k) => {
          if(v !== 0 && 
            item[k - 1] === v && 
            item[k - 2] === v &&
            item[k + 1] === v &&
            item[k + 2] === v) {
              return true;
          }
        })
        if(temp) {
          totalScore++;
        }
      });  
      return totalScore;
    },

    /**
     * 从新开始游戏
     */
    reStart() {
      const { canvas, ctx } = this;

      ctx.clearRect(0, 0, canvas.width, canvas.height);
      this.drawChessBoard();
      this.initChessboardMatrix();
      this.listenDropChessman();      
      this.isBlack = true;
      this.tips = '黑方的回合';
    },

    /**
     * 悔棋
     */
    unDo() {      
      if(!this.preStep) return;  // 没有落子
      if(this.gameOver) return;  // 游戏结束
      const { ctx, chessBoardMatrix } = this;
      const { x, y, canUndo } = this.preStep;      
      const { padding, color } = this.chessBoardStyle;
      // const _x = (x + 1) * padding + 0.5;  // 上一步棋子所在的交叉点x
      // const _y = (y + 1) * padding + 0.5;  // 上一步棋子所在的交叉点y
      const _x = (x + 1) * padding;  // 上一步棋子所在的交叉点x
      const _y = (y + 1) * padding;  // 上一步棋子所在的交叉点y

      if(!canUndo) return;
      ctx.clearRect((x + 0.5) * padding + 0.5, (y + 0.5) * padding + 0.5, padding, padding);
      chessBoardMatrix[x][y] = 0;  // 重置矩阵到上一步
      this.isBlack = !this.isBlack;  // 返回上一角色      
      this.preStep.canUndo = false;  // 不允许连续悔棋
      ctx.save();
      ctx.strokeStyle = color;            
      ctx.beginPath();    
      this.drawLine(ctx, { x: _x, y: _y }, { x: _x, y: _y - padding * 0.6 });
      this.drawLine(ctx, { x: _x, y: _y }, { x: _x, y: _y + padding * 0.6 });
      this.drawLine(ctx, { x: _x, y: _y }, { x: _x - padding * 0.6, y: _y });
      this.drawLine(ctx, { x: _x, y: _y }, { x: _x + padding * 0.6, y: _y });
      // ctx.moveTo(_x, _y);
      // ctx.lineTo(_x, _y - padding * 0.6); // 0.6是为了覆盖更多像素，0.5时会有瑕疵
      // ctx.moveTo(_x, _y);
      // ctx.lineTo(_x, _y + padding * 0.6);
      // ctx.moveTo(_x, _y);
      // ctx.lineTo(_x - padding * 0.6, _y);
      // ctx.moveTo(_x, _y);
      // ctx.lineTo(_x + padding * 0.6, _y);
      ctx.stroke();       
      ctx.restore();  // 将样式属性出栈
    },

    /**
     * 撤销悔棋
     */
    cancelUndo() {            
      if(!this.preStep) return;  // 没有落子
      if(this.gameOver) return;  // 游戏结束
      const { x, y, isBlack, canCancelUndo, canUndo} = this.preStep;      
      const { ctx, chessBoardMatrix } = this;
      const { padding } = this.chessBoardStyle;

      if(canUndo) return;
      this.preStep.canUndo = true; // 撤销悔棋后又可以悔棋
      this.drawChessman((x + 1) * padding, (y + 1) * padding);
      this.isBlack = !this.isBlack;
      chessBoardMatrix[x][y] = isBlack ? 1 : 2;
    }
  },
  
  created() {
    this.initChessboardMatrix();
  },

  mounted() {
    this.initCanvas();        
    this.listenDropChessman();
  },
}
</script>

<style scope>
  .chessBoard {     
    --main-color: #666;  
    position: absolute; 
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);    
    text-align: center;    
    user-select:none;
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
    margin: 60px;
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
    background-color: #EFEFEF;
  }

  canvas {    
    box-shadow: 2px 2px 4px 8px #eee;
  }
</style>
