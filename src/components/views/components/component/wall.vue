<template>
  <div class='divWall' >
    <div v-for="(row,rowIndex) in wallArray" :key="rowIndex" >
      <div v-for="(item,colIndex) in row" :key="colIndex"
        class="square-block" :style="{width:(borderWidth+10)+'px',height:(borderWidth+10)+'px'}">
            <div class="inner-seat"
              :class="styleObject(wallArray[rowIndex][colIndex])"
              @click="setDoor(rowIndex,colIndex,wallArray[rowIndex][colIndex])">
            </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {

  props: ['isShow', 'startTopIndex', 'startLeftIndex', 'wallWidth', 'wallHeight', 'borderWidth', 'doorArea'],

  computed: {

    divWidth() {
      return this.wallWidth;
    },
    divHeight() {
      return this.wallHeight;
    },
    wallArray() {
      return this.renderWall();
    },

    styleObject() {
      return (type) => {
        switch (type) {
        case 'topleft':
          return 'top-left';
        case 'topright':
          return 'top-right';
        case 'bottomleft':
          return 'bottom-left';
        case 'bottomright':
          return 'bottom-right';
        case 'horizontalsectionleft': case 'horizontalsectionright':
          return 'horizontal-section';
        case 'verticalsectionup': case 'verticalsectiondown':
          return 'vertical-section';
        case 'door':
          return 'door';
        default:
          return 'none';
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

    renderWall() {
      const columns = this.wallWidth;
      const rows = this.wallHeight;

      // 确定第一个环的左上开始位置(最外环为第一个环)
      const startRowIndex = 0; // 根据布局大小计算环开始的行位置
      const startColIndex = 0; // 根据布局大小计算环开始的列位置

      this.initRectArray();


      const oldArray = this.rectArray.slice();

      this.hLayout(rows, columns, startRowIndex, startColIndex, oldArray);
      // 四个角
      oldArray[0][0] = 'topleft';
      oldArray[0][columns - 1] = 'topright';
      oldArray[rows - 1][0] = 'bottomleft';
      oldArray[rows - 1][columns - 1] = 'bottomright';

      if (this.doorArea && this.doorArea.length > 0) {
        for (let i = 0; i < this.doorArea.length; i += 1) {
          oldArray[this.doorArea[i].top][this.doorArea[i].left] = 'none';
        }
      }
      this.rectArray = oldArray;
      return this.rectArray;
    },
    /**
       * 根据横向和纵向的座位数画环
       * rows:横向座位数
       * columns:纵向座位数
       * startRowIndex:左上角开始行位置
       * startColIndex:左上角开始的列位置
       * oldArray:需要标注画环的二维数组
       */
    hLayout(rows, columns, startRowIndex, startColIndex, oldArray) {
      for (let i = 0; i < rows - 1; i += 1) {
        oldArray[startRowIndex + i][startColIndex] = 'horizontalsectionleft';
        oldArray[startRowIndex + i][startColIndex + columns - 1] = 'horizontalsectionright';
      }
      for (let j = 0; j < columns; j += 1) {
        oldArray[startRowIndex + rows - 1][startColIndex + j] = 'verticalsectiondown';
        oldArray[startRowIndex][startColIndex + j] = 'verticalsectionup';
      }
      return oldArray;
    },

    setDoor(rowIndex, colIndex, type) {
      // this.wallArray[rowIndex][colIndex] = 'door';
      // this.wallArray[rowIndex][colIndex - 1] = 'door';
      this.$emit('addDoor', rowIndex, colIndex, type);
    },

    initRectArray() {
      // 初始化布局
      if (this.divWidth) {
        const rectArray = Array(this.divHeight).fill('none')
          .map(() => Array(this.divWidth).fill('none'));
        this.rectArray = rectArray;
      }
    },
  },

};
</script>

<style scoped>
  .divWall{
    position: absolute;
    pointer-events: none;
  }
  /*画布中小正方形 */
  .square-block{
    float:left;
    display: flex;
    justify-content: center;
    align-items: center;
    /* background: #EBEBEB; */
  }
  .inner-seat{
    width:100%;
    height:100%;
    cursor: pointer;
  }
  .top-left{
    background: url('../../../../assets/image/web/seat/wall-top-left.svg')
    center center no-repeat;
    background-size: 100% 100%;
    pointer-events: auto
  }
  .top-right{
    background: url('../../../../assets/image/web/seat/wall-top-right.svg')
    center center no-repeat;
    background-size: 100% 100%;
    pointer-events: auto
  }
  .bottom-left{
    background: url('../../../../assets/image/web/seat/wall-bottom-left.svg')
    center center no-repeat;
    background-size: 100% 100%;
    pointer-events: auto
  }
  .bottom-right{
    background: url('../../../../assets/image/web/seat/wall-bottom-right.svg')
    center center no-repeat;
    background-size: 100% 100%;
    pointer-events: auto
  }
  .horizontal-section{
    background: url('../../../../assets/image/web/seat/wall-horizontal-middle.svg')
    center center no-repeat;
    background-size: 100% 100%;
    pointer-events: auto
  }
  .vertical-section{
      opacity: 1.0;
      background: url('../../../../assets/image/web/seat/wall-vertical-middle.svg')
      center center no-repeat;
      background-size: 100% 100%;
      pointer-events: auto
  }
    .door{
      opacity: 1.0;
      background: url('../../../../assets/image/web/seat/door.svg')
      center center no-repeat;
      background-size: 100% 100%;
      pointer-events: auto
  }
  .none{
      opacity: 0.0;
       background-size: 100% 100%;
       background: white;
  }
</style>>
