<template>
<div :class="dragClass" >
     <!--根据相关参数画模型-->
     <!-- <div class="mid-liney" v-if="reqType!==1"></div>
     <div class="mid-linex" v-if="reqType!==1"></div> -->
     <div v-for="(row,rowIndex) in seatModelArray" :key="rowIndex" :v-model="seatModelArray">
       <div v-for="(item,colIndex) in row" :key="colIndex"
        :class="[preClass,seatStyle(seatModelArray[rowIndex][colIndex].type
        ,seatModelArray[rowIndex][colIndex].orientation)]"
         :style="{width:gridSize+'px',height:gridSize+'px'
        ,border:'white '+ doubleBorder/2+'px solid'}" class="inner-seat">
      </div>
    </div>
     <!--根据相关seats 复原座位-->
     <!--画小图-->
</div>
</template>

<script>
export default {
  /** 画模型 */
  props: {
    gridSize: Number, // 单元格大小
    seatModelArrayWidth: Number, // 宽
    seatModelArrayHeight: Number, // 高
    modelType: String, // 模型类型
    orientation: String, // 朝向
    circleNum: Number, // 环数
    // grid边框宽度
    doubleBorder: {
      type: Number,
      default: 10,
    },
    topIndex: {
      type: Number,
      default: 2,
    },
    leftIndex: {
      type: Number,
      default: 2,
    },
    // 座位类型 默认1 普通座位
    seatType: {
      type: Number,
      default: 1,
    },
    // seatsArray: { type: Array, default: Array }, // 座位数组
    reqType: {
      type: Number,
      default: 1,
    }, // 请求类型 （1画模型，2画预览）
  },
  computed: {
    dragClass() {
      return this.reqType !== 4 ? (this.reqType === 1? 'drag':'unset-drag') : 'undrag';
    },

    preClass() {
      return this.reqType !== 4 ? 'square-block' : 'pre-square-block';
    },
    divWidth() {
      return this.seatModelArrayWidth;
    },
    divHeight() {
      return this.seatModelArrayHeight;
    },
    seatModelArray() {
      return this.renderSeat();
    },

    // seatStyle() {
    //   return (type) => {
    //     if (type !== 1) {
    //       return 'no-seat';
    //     }
    //     if (this.reqType !== 1) {
    //       return 'pre-normal-seat';
    //     }
    //     return 'normal-seat';
    //   };
    // },

     seatStyle() {
      return (type, orientation) => {
        // 不是座位
         if (type !== 1) {
          return 'no-seat';
        }
        //预览状态
        if(this.reqType===4){
          return 'pre-normal-seat';
        }

       switch (orientation) {
          case 'l':
            return this.reqType===1?'normal-seat-left':(this.reqType===2?'normal-seat-selected-left':'normal-seat-unset-left');
          case 'r':
            return this.reqType===1?'normal-seat-right':(this.reqType===2?'normal-seat-selected-right':'normal-seat-unset-right');
          case 'u':
            return this.reqType===1?'normal-seat-up':(this.reqType===2?'normal-seat-selected-up':'normal-seat-unset-up');
          case 'd':
            return this.reqType===1?'normal-seat-down':(this.reqType===2?'normal-seat-selected-down':'normal-seat-unset-down');
          default:
            return this.reqType===1?'normal-seat-down':(this.reqType===2?'normal-seat-selected-down':'normal-seat-unset-down');
          }

      };
    },

  },


  data() {
    return {
      rectArray: [],
      emptyFlag: 0,
    };
  },
  methods: {
    // 渲染座位区域 目前只有U型
    renderSeat() {
      const columns = this.seatModelArrayWidth;
      const rows = this.seatModelArrayHeight;
      const { orientation } = this;
      const { circleNum } = this;
      // 确定第一个环的左上开始位置(最外环为第一个环)
      const startRowIndex = 0; // 根据布局大小计算环开始的行位置
      const startColIndex = 0; // 根据布局大小计算环开始的列位置

      this.initRectArray();

      let oldArray = this.rectArray.slice();
      // oldArray=this.hLayout(rows+4,columns+4,startRowIndex-2,startColIndex-2,oldArray,2);
      // validate(); //校验逻辑
      // 如果是U型
      // 朝向上或者下
      // 环数1 则 横向位数 不能小于3 纵向不能小于2  且都不能大于20
      // 环数2 则 横向位数 不能小于5 纵向不能小于3  且都不能大于20
      // 环数3 则 横向位数 不能小于7 纵向不能小于4  且都不能大于20
      // 朝向左或者右
      // 环数1 则 横向位数 不能小于2 纵向不能小于3  且都不能大于20
      // 环数2 则 横向位数 不能小于3 纵向不能小于5  且都不能大于20
      // 环数3 则 横向位数 不能小于4 纵向不能小于7  且都不能大于20
      // 校验逻辑也可以不写在这里直接 用rule 规则取校验
      // 目前只考虑一个环
      if (this.modelType === 'uShape') {
        if (circleNum === 1) {
          // 画环
          oldArray = this.hLayout(rows, columns, startRowIndex, startColIndex, oldArray);
          // 根据方向去掉多余的座位
          this.rectArray = this.uLayout(startRowIndex, startColIndex,
            columns, rows, oldArray, orientation);
        } else if (circleNum === 2) {
          // 画第一个环
          oldArray = this.hLayout(rows, columns, startRowIndex, startColIndex, oldArray);
          // 画第二个环
          oldArray = this.hLayout(rows - 2, columns - 2, startRowIndex + 1,
            startColIndex + 1, oldArray);
          // 去掉第一个环多余座位
          oldArray = this.uLayout(startRowIndex, startColIndex, columns,
            rows, oldArray, orientation);
          // 去掉第二个环多余座位
          this.rectArray = this.uLayout(startRowIndex + 1, startColIndex + 1,
            columns - 2, rows - 2, oldArray, orientation, 2);
        } else if (circleNum === 3) {
          // 画第一个环
          oldArray = this.hLayout(rows, columns, startRowIndex, startColIndex, oldArray);
          // 画第二个环
          oldArray = this.hLayout(rows - 2, columns - 2, startRowIndex + 1,
            startColIndex + 1, oldArray);
          // 画第三个环
          oldArray = this.hLayout(rows - 4, columns - 4, startRowIndex + 2,
            startColIndex + 2, oldArray);
          // 去掉第一个环多余座位
          oldArray = this.uLayout(startRowIndex, startColIndex, columns,
            rows, oldArray, orientation);
          // 去掉第二个环多余座位
          oldArray = this.uLayout(startRowIndex + 1, startColIndex + 1,
            columns - 2, rows - 2, oldArray, orientation, 2);
          // 去掉第三个环多余座位
          this.rectArray = this.uLayout(startRowIndex + 2, startColIndex + 2,
            columns - 4, rows - 4, oldArray, orientation, 3);
        }
      } else if (this.modelType === 'gyrationShape') {
        // 回型
        oldArray = this.hLayout(rows, columns, startRowIndex, startColIndex, oldArray);
        // // 去掉四个角
        oldArray[0][0].type = this.emptyFlag;
        oldArray[0][columns - 1].type = this.emptyFlag;
        oldArray[rows - 1][0].type = this.emptyFlag;
        oldArray[rows - 1][columns - 1].type = this.emptyFlag;
        this.rectArray = oldArray;
      } else if (this.modelType === 'deskShape') {
        // 课桌型
        this.rectArray = this.tableLayout(rows, columns, startRowIndex, startColIndex, oldArray);
      }
      return this.rectArray;
    },

    /** 根据朝向 去掉环中多余的座位
       * rows:横向座位数
       * columns:纵向座位数
       * startRowIndex:左上角开始行位置
       * startColIndex:左上角开始的列位置
       * oldArray:需要标注画环的二维数组
       * orientation:朝向
       * circleNum:当前处理第几个环（从外到里依次1、2、3）
       */
    uLayout(startRowIndex, startColIndex, columns, rows, oldArray, orientation, circleNum = 1) {
      // 向上
      if (orientation === 'u') {
        for (let i = 1; i < columns - 1; i += 1) {
          oldArray[startRowIndex][startColIndex + i].type = this.emptyFlag;
        }
        oldArray[startRowIndex + rows - 1][startColIndex].type = this.emptyFlag;
        oldArray[startRowIndex + rows - 1][startColIndex + columns - 1].type = this.emptyFlag;
        // 最上变两边的座位改变朝向
        oldArray[startRowIndex][startColIndex].orientation = 'r';
        oldArray[startRowIndex][startColIndex + columns - 1].orientation = 'l';
        if (circleNum === 2) {
          oldArray[startRowIndex - 1][startColIndex].type = this.seatType;
          oldArray[startRowIndex - 1][startColIndex + columns - 1].type = this.seatType;
        } else if (circleNum === 3) {
          oldArray[startRowIndex - 1][startColIndex].type = this.seatType;
          oldArray[startRowIndex - 1][startColIndex + columns - 1].type = this.seatType;
          oldArray[startRowIndex - 2][startColIndex].type = this.seatType;
          oldArray[startRowIndex - 2][startColIndex + columns - 1].type = this.seatType;
        }
      } else if (orientation === 'd') {
        // 向下
        for (let i = 1; i < columns - 1; i += 1) {
          oldArray[startRowIndex + rows - 1][startColIndex + i].type = this.emptyFlag;
        }
        oldArray[startRowIndex][startColIndex].type = this.emptyFlag;
        oldArray[startRowIndex][startColIndex + columns - 1].type = this.emptyFlag;
        // 最底端两边的座位改变朝向
        oldArray[startRowIndex + rows - 1][startColIndex].orientation = 'r';
        oldArray[startRowIndex + rows - 1][startColIndex + columns - 1].orientation = 'l';

        if (circleNum === 2) {
          oldArray[startRowIndex + rows][startColIndex].type = this.seatType;
          oldArray[startRowIndex + rows][startColIndex + columns - 1].type = this.seatType;
        } else if (circleNum === 3) {
          oldArray[startRowIndex + rows][startColIndex].type = this.seatType;
          oldArray[startRowIndex + rows][startColIndex + columns - 1].type = this.seatType;
          oldArray[startRowIndex + rows + 1][startColIndex].type = this.seatType;
          oldArray[startRowIndex + rows + 1][startColIndex + columns - 1].type = this.seatType;
        }
      } else if (orientation === 'l') {
        // 向左
        for (let i = 1; i < rows - 1; i += 1) {
          oldArray[startRowIndex + i][startColIndex].type = this.emptyFlag;
        }
        oldArray[startRowIndex][startColIndex + columns - 1].type = this.emptyFlag;
        oldArray[startRowIndex + rows - 1][startColIndex + columns - 1].type = this.emptyFlag;

        if (circleNum === 2) {
          oldArray[startRowIndex][startColIndex - 1].type = this.seatType;
          oldArray[startRowIndex + rows - 1][startColIndex - 1].type = this.seatType;
        } else if (circleNum === 3) {
          oldArray[startRowIndex][startColIndex - 1].type = this.seatType;
          oldArray[startRowIndex + rows - 1][startColIndex - 1].type = this.seatType;
          oldArray[startRowIndex][startColIndex - 2].type = this.seatType;
          oldArray[startRowIndex + rows - 1][startColIndex - 2].type = this.seatType;
        }
      } else if (orientation === 'r') {
        // 向右
        for (let i = 1; i < rows - 1; i += 1) {
          oldArray[startRowIndex + i][startColIndex + columns - 1].type = 0;
        }
        oldArray[startRowIndex][startColIndex].type = this.emptyFlag;
        oldArray[startRowIndex + rows - 1][startColIndex].type = this.emptyFlag;

        if (circleNum === 2) {
          oldArray[startRowIndex][startColIndex + columns].type = this.seatType;
          oldArray[startRowIndex + rows - 1][startColIndex + columns].type = this.seatType;
        } else if (circleNum === 3) {
          oldArray[startRowIndex][startColIndex + columns].type = this.seatType;
          oldArray[startRowIndex + rows - 1][startColIndex + columns].type = this.seatType;
          oldArray[startRowIndex][startColIndex + columns + 1].type = this.seatType;
          oldArray[startRowIndex + rows - 1][startColIndex + columns + 1].type = this.seatType;
        }
      }
      return oldArray;
    },

    /**
       * 根据横向和纵向的座位数画环
       * rows:横向座位数
       * columns:纵向座位数
       * startRowIndex:左上角开始行位置
       * startColIndex:左上角开始的列位置
       * oldArray:需要标注画环的二维数组
       * setValue:设置环位置的数组坐标占位符
       */
    hLayout(rows, columns, startRowIndex, startColIndex, oldArray, setValue = 1) {
      for (let i = 0; i < rows - 1; i += 1) {
        oldArray[startRowIndex + i][startColIndex] = { type: setValue, orientation: 'r' };
        oldArray[startRowIndex + i][startColIndex + columns - 1] = { type: setValue, orientation: 'l' };
      }
      for (let j = 0; j < columns; j += 1) {
        oldArray[startRowIndex + rows - 1][startColIndex + j] = { type: setValue, orientation: 'u' };
        oldArray[startRowIndex][startColIndex + j] = { type: setValue, orientation: 'd' };
      }
      return oldArray;
    },
    /**
       * 根据横向和纵向的座位数布局课桌型
       * rows:横向座位数
       * columns:纵向座位数
       * startRowIndex:左上角开始行位置
       * startColIndex:左上角开始的列位置
       * oldArray:需要标注画环的二维数组
       * setValue:设置环位置的数组坐标占位符
       */
    tableLayout(rows, columns, startRowIndex, startColIndex, oldArray, setValue = 1) {
      for (let i = 0; i < rows; i += 1) {
        for (let j = 0; j < columns; j += 1) {
          oldArray[startRowIndex + i][startColIndex + j] = { type: setValue, orientation: 'd' };
        }
      }
      return oldArray;
    },
    initRectArray() {
      // 初始化布局
      if (this.divWidth) {
        const rectArray = Array(this.divHeight).fill({ type: this.emptyFlag, orientation: '' })
          .map(() => Array(this.divWidth).fill({ type: this.emptyFlag, orientation: '' }));
        this.rectArray = rectArray;
      }
    },

  },
  mounted() {
    this.$emit('initSeatModelArray', this.rectArray);
  },

};
</script>

<style lang="scss"  scoped>

  /*画布中小正方形 */
  .square-block{
    float:left;
    display: flex;
    justify-content: center;
    align-items: center;
    background: white;
    opacity: 0.5;
    border-color:white
  }
  .pre-square-block{
    float:left;
    display: flex;
    justify-content: center;
    align-items: center;
    background:white;
  }
  .drag{
      position: absolute;
      top: 0;
      left: 0;
      pointer-events: none;
     border-radius: 2px;
     background-size: cover;
     background: rgba(13, 175, 156, 0.15);
  }
    .unset-drag{
      position: absolute;
      top: 0;
      left: 0;
      pointer-events: none;
     border-radius: 2px;
     background-size: cover;
     background: rgba(253, 87, 81, 0.15);
  }
  .undrag{
     position: absolute;
     pointer-events: none;
     background-size: cover;
  }
  .inner-seat{
    width:100%;
    height:100%;
    cursor: pointer;
  }
   .normal-seat-left{
     background: url('../../../../assets/image/web/seat/normal-seat-left.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .normal-seat-right{
    background: url('../../../../assets/image/web/seat/normal-seat-right.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
   .normal-seat-up{
   background: url('../../../../assets/image/web/seat/normal-seat-up.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .normal-seat-down{
    background: url('../../../../assets/image/web/seat/normal-seat-down.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
     .normal-seat-selected-left{
     background: url('../../../../assets/image/web/seat/normal-seat-selected-left.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .normal-seat-selected-right{
    background: url('../../../../assets/image/web/seat/normal-seat-selected-right.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
   .normal-seat-selected-up{
   background: url('../../../../assets/image/web/seat/normal-seat-selected-up.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .normal-seat-selected-down{
    background: url('../../../../assets/image/web/seat/normal-seat-selected-down.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
     .normal-seat-unset-left{
     background: url('../../../../assets/image/web/seat/normal-seat-unset-left.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .normal-seat-unset-right{
    background: url('../../../../assets/image/web/seat/normal-seat-unset-right.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
   .normal-seat-unset-up{
   background: url('../../../../assets/image/web/seat/normal-seat-unset-up.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .normal-seat-unset-down{
    background: url('../../../../assets/image/web/seat/normal-seat-unset-down.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
  .pre-normal-seat{
    background: #7AA4CA;
    background-size: 100% 100%;
    border-radius: 2px;
  }
 .no-seat{
    background-size: 100% 100%;
    line-height: 42px;
    opacity: 0.0;
  }
  .unselected-seat{
    background-size: 100% 100%;
  }
</style>
