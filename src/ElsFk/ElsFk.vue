<template>
  <div>
    <div ref="game" id="game" class="game">
      <div
        v-for="(items, itemsIndex) in gameData" :key="itemsIndex">
        <div
          v-for="(item, itemIndex) in items"
          :class="{'none': item === 0, 'current': item === 1, 'done': item === 2}"
          :style="{'top': itemsIndex * 20 + 'px', 'left': itemIndex * 20 + 'px'}"
          :key="itemIndex">
          </div>
      </div>
    </div>
    <div class="next" id="next">
      <div
        v-for="(items, itemsIndex) in nextData" :key="itemsIndex">
        <div
          v-for="(item, itemIndex) in items"
          :class="{'none': item === 0, 'current': item === 1, 'done': item === 2}"
          :style="{'top': itemsIndex * 20 + 'px', 'left': itemIndex * 20 + 'px'}"
          :key="itemIndex">
          </div>
      </div>
    </div>
    <div class="info">
      <!-- <div>已用时：<span id="time">0</span>秒</div> -->
      <div>已得分：<span id="score">{{score}}</span>分</div>
      <div>游戏状态：<span id="gameState">{{gameState}}</span></div>
    </div>
  </div>
</template>
<script>
export default {
  data () {
    return {
      nextData: [ // 下一个方块
        [1, 1, 1, 1],
        [0, 0, 0, 0],
        [0, 0, 0, 0],
        [0, 0, 0, 0]
      ],
      gameData: [ // 方块界面
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
      ],
      nextDiv: null,
      gameDiv: null,
      nowFallDownData: [ // 当前下落方块数据
        [1, 1, 1, 1],
        [0, 0, 0, 0],
        [0, 0, 0, 0],
        [0, 0, 0, 0]
      ],
      squareMatrixData: [ // 7种方块数据
        [
          [
            [1, 1, 1, 1],
            [0, 0, 0, 0],
            [0, 0, 0, 0],
            [0, 0, 0, 0]
          ]
        ],
        [
          [
            [1, 1, 1, 0],
            [1, 0, 0, 0],
            [0, 0, 0, 0],
            [0, 0, 0, 0]
          ]
        ],
        [
          [
            [1, 1, 1, 0],
            [0, 0, 1, 0],
            [0, 0, 0, 0],
            [0, 0, 0, 0]
          ]
        ],
        [
          [
            [1, 1, 1, 0],
            [0, 1, 0, 0],
            [0, 0, 0, 0],
            [0, 0, 0, 0]
          ]
        ],
        [
          [
            [1, 1, 0, 0],
            [1, 1, 0, 0],
            [0, 0, 0, 0],
            [0, 0, 0, 0]
          ]
        ],
        [
          [
            [1, 1, 0, 0],
            [0, 1, 1, 0],
            [0, 0, 0, 0],
            [0, 0, 0, 0]
          ]
        ],
        [
          [
            [0, 1, 1, 0],
            [1, 1, 0, 0],
            [0, 0, 0, 0],
            [0, 0, 0, 0]
          ]
        ]
      ],
      origin: { // 方块位置原点
        x: 0,
        y: 0
      },
      timer: null, // 下落计时器
      interval: 400, // 下落速度越小越快
      start: false,
      gameState: '开始',
      score: 0,
      shape: 0, // 方块形状编号
      direction: 0, // 方块方向编号
      nextShap: 0, // 下一个方块形状编号
      nextDirection: 0 // 下一个方块方向编号
    }
  },
  created () {
    this.buildSquareMatrixData()
    this.createNextSquare()
    this.copyNextSquare()
    this.createNextSquare()
    this.bindKeyEvent()
  },
  methods: {
    bindKeyEvent () {
      document.onkeydown = (e) => {
        if (e.keyCode === 32) {
          // space 游戏开始
          this.squareFall()
        }
        if (this.start) {
          if (e.keyCode === 38) {
            // up
            this.squareDirectionChange()
          } else if (e.keyCode === 39) {
            // right
            this.squareMove('right')
          } else if (e.keyCode === 40) {
            // down
            this.squareMove('down')
          } else if (e.keyCode === 37) {
            // left
            this.squareMove('left')
          }
        }
      }
    },
    createNextSquare () {
      this.nextShap = Math.floor(Math.random() * this.squareMatrixData.length)
      this.nextDirection = Math.floor(Math.random() * 3 + 1)
      this.nextData = this.squareMatrixData[this.nextShap][this.nextDirection]
    },
    copyNextSquare () {
      this.nowFallDownData = this.nextData
      this.shape = this.nextShap
      this.direction = this.nextDirection
      this.origin.x = 0
      this.origin.y = (this.gameData[0].length / 2) - 2
    },
    squareFall () { // 游戏开始
      if (this.timer) {
        clearInterval(this.timer)
        this.timer = null
        this.start = false
        this.gameState = '游戏暂停'
      } else {
        this.start = true
        this.gameState = '游戏进行中'
        this.timer = setInterval(() => {
          let squareFlag = this.squareMove('down')
          if (!squareFlag) {
            // 固定方块
            this.updateGameData(2)
            // 检测消除,更新分数
            this.checkLine()
            // 复制下一个方块,重置方块原点
            this.copyNextSquare()
            // 做游戏结束判断
            if (!this.isValid(this.origin)) {
              // 清除clearInterval
              clearInterval(this.timer)
              this.timer = null
              this.gameState = '游戏结束'
            }
            // 生成新的下一个方块
            this.createNextSquare()
          }
        }, this.interval)
      }
    },
    squareMove (moveType) { // 方块移动控制
      let pos = {x: 0, y: 0}
      if (moveType === 'down') {
        pos.x = this.origin.x + 1
        pos.y = this.origin.y
      } else if (moveType === 'left') {
        pos.x = this.origin.x
        pos.y = this.origin.y - 1
      } else if (moveType === 'right') {
        pos.x = this.origin.x
        pos.y = this.origin.y + 1
      }
      if (this.isValid(pos)) {
        this.updateGameData(0)
        this.origin.x = pos.x
        this.origin.y = pos.y
        this.updateGameData(1)
      } else {
        return false // 接触障碍物后返回false,固定方块
      }
      return true
    },
    squareDirectionChange () { // 方块方向变化
      this.changeDirection(1)
      this.nowFallDownData = this.squareMatrixData[this.shape][this.direction]
      if (this.isValid(this.origin)) {
        this.updateGameData(0)
        this.updateGameData(1)
      } else {
        this.changeDirection(-1)
        this.nowFallDownData = this.squareMatrixData[this.shape][this.direction]
      }
    },
    updateGameData (updateType) { // 更新场景数据，0代表清除，1表示动态加入，2表示固实加入
      for (let i = 0; i < this.nowFallDownData.length; i++) {
        for (let j = 0; j < this.nowFallDownData[0].length; j++) {
          if (this.checkDeany(this.origin, i, j)) {
            if (updateType === 0) {
              this.$set(this.gameData[this.origin.x + i], this.origin.y + j, 0)
            } else if (updateType === 1) {
              this.$set(this.gameData[this.origin.x + i], this.origin.y + j, this.nowFallDownData[i][j])
            } else {
              if (this.nowFallDownData[i][j] === 1) {
                this.$set(this.gameData[this.origin.x + i], this.origin.y + j, 2)
              }
            }
          }
        }
      }
    },
    checkDeany (pos, x, y) { // 检测方块点是否合法
      if (pos.x + x >= 0 && pos.x + x < this.gameData.length && pos.y + y >= 0 && pos.y + y < this.gameData[0].length && (this.gameData[pos.x + x][pos.y + y] !== 2)) {
        return true
      } else {
        return false
      }
    },
    checkLine () { // 检测是否有可消除的整行 根据消除情况重新生成场景数据，更新分数
      let clearCount = 0
      let tempGameData = []
      for (let item of this.gameData) {
        if (item.reduce((accumulator, currentValue) => accumulator + currentValue) >= this.gameData[0].length * 2) {
          tempGameData.unshift([].fill.call(new Array(this.gameData[0].length), 0))
          clearCount += 1
        } else {
          tempGameData.push(item)
        }
      }
      if (clearCount > 0) {
        this.gameData = tempGameData
        this.updateScore(clearCount)
      }
    },
    updateScore (clearCount) {
      let prizePoint = 0
      if (clearCount >= 1 && clearCount < 4) {
        prizePoint = clearCount * clearCount
      } else {
        prizePoint = clearCount * 6
      }
      this.score += prizePoint
    },
    isValid (pos) { // 检测数据是否合法
      for (let i = 0; i < this.nowFallDownData.length; i++) {
        for (let j = 0; j < this.nowFallDownData[0].length; j++) {
          if (this.nowFallDownData[i][j] !== 0) {
            if (!this.checkDeany(pos, i, j)) {
              return false
            }
          }
        }
      }
      return true
    },
    buildSquareMatrixData () { // 生成下落方块形状转换方向后的数据
      let tempData = []
      for (let i = 0; i < this.squareMatrixData.length; i++) {
        for (let j = 1; j < 4; j++) {
          tempData = this.transformBlock(1, this.squareMatrixData[i][j - 1])
          tempData = this.formatBlock(tempData)
          this.squareMatrixData[i].push(tempData)
        }
      }
    },
    formatBlock (block) { // 整理方块矩阵数据，使方块位于矩阵左上角
      block = this.removeTopBlack(block)
      block = this.transformBlock(1, block)
      block = this.removeTopBlack(block)
      block = this.transformBlock(0, block)
      return block
    },
    removeTopBlack (block) { // 清除方块矩阵顶部空行
      for (let i = 0; i < 4; i++) {
        if (block[0].reduce((accumulator, currentValue) => accumulator + currentValue) < 1) { // 是空行
          block.shift()
          block.push([0, 0, 0, 0])
        }
      }
      return block
    },
    transformBlock (directionType = 0, block) { // 旋转方块数据
      let tempData = []
      for (let i = 0; i < 4; i++) {
        tempData.push([])
        for (let j = 0; j < 4; j++) {
          if (directionType === 0) {
            tempData[i].push(block[j][3 - i])
          } else {
            tempData[i].push(block[3 - j][i])
          }
        }
      }
      return tempData
    },
    changeDirection (value) { // 改变方块方向编号
      this.direction += value
      if (this.direction > 3) {
        this.direction = 0
      } else if (this.direction < 0) {
        this.direction = 3
      }
    }
  },
  mounted () {
  }
}
</script>
<style lang="scss" scoped>
.game {
  position: absolute;
  top: 10px;
  left: 10px;
  width: 200px;
  height: 400px;
  background: #f2faff;
  border-left: 1px solid blue;
  border-right: 1px solid blue;
  border-bottom: 1px solid blue;
}
.next {
  position: absolute;
  top: 10px;
  left: 250px;
  width: 80px;
  height: 80px;
  background: #f2faff;
  border: 1px solid blue;
}
.info {
  position: absolute;
  top: 100px;
  left: 250px;
}
.none, .current, .done {
  position: absolute;
  width: 20px;
  height: 20px;
  box-sizing: border-box;
}
.none {
  background: #f2faff;
}
.current {
  background: pink;
  border: 1px solid red;
}
.done {
  background: gray;
  border: 1px solid black;
}
</style>
