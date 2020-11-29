<template>
<div :class="[dragClass, transformClass]">
     <!--会议桌-->
     <div v-if="facilityType==='table'" :class='divTableClass'
      :style="{width:facilityWidth*(gridSize+doubleBorder)+'px',
      height:facilityHeight*(gridSize+doubleBorder)+'px'}" >
    </div>
     <!--主席台-->
    <div v-else-if="facilityType === 'rostrum'"  class='divRostrumBox'>
      <div class='divRostrumSeats'>
        <div :style="{width:(facilityWidth-holdSeatNum)/2*(gridSize+doubleBorder)+'px',
        height:(gridSize+doubleBorder)+'px'}"></div>
        <div v-for="(itme,Index) in holdSeatNum" :key="Index"  :class="rostrumSeatClass" :value="getArray[Index]"
             :style="{width:gridSize+'px',height:gridSize+'px'}">
        </div>
      </div>
        <div :class='divRostrumClass' :style="{width:facilityWidth*(gridSize+doubleBorder)+'px',
         height:(facilityHeight-1)*(gridSize+doubleBorder)+'px'}"></div>
    </div>
     <!--门-->
    <div v-else-if="facilityType === 'door'"
        :style="{width:(gridSize)*facilityWidth+'px',height:(gridSize)*facilityHeight+'px'}" :class ='divDoorClass'>
        <!-- <img src='../../../../assets/image/web/seat/real-door.svg' width="100%" height= "100%"/> -->
    </div>
     <!--屏幕-->
    <div v-else-if="facilityType === 'mainScreen'" class='divMainScreen'>
       <img :src="reqType === 1?require('../../../../assets/image/web/seat/screen-selected-left.svg'):
      (reqType === 2?require('../../../../assets/image/web/seat/screen-left.svg'):
      require('../../../../assets/image/web/seat/screen-unset-left.svg'))" >
       <img v-for="(itme,index) in facilityWidth-2" :key="index"
       :src="reqType === 1?require('../../../../assets/image/web/seat/screen-selected-middle.svg'):
      (reqType === 2?require('../../../../assets/image/web/seat/screen-middle.svg'):
      require('../../../../assets/image/web/seat/screen-unset-middle.svg'))">

       <img  :src="reqType === 1?require('../../../../assets/image/web/seat/screen-selected-right.svg'):
      (reqType === 2?require('../../../../assets/image/web/seat/screen-right.svg'):
      require('../../../../assets/image/web/seat/screen-unset-right.svg'))">
    </div>
    <div v-else>
      请选择设施
    </div>
</div>
</template>

<script>
export default {
  /** 画模型 */
  props: {
    gridSize: Number, // 单元格大小
    facilityWidth: Number, // 宽
    facilityHeight: Number, // 高
    facilityType: String, // 设施类型
    facilityOrientation: String, // 朝向
    // grid边框宽度
    doubleBorder: {
      type: Number,
      default: 10,
    },
    // 生成时候的起始位置
    topIndex: {
      type: Number,
      default: 2,
    },
    leftIndex: {
      type: Number,
      default: 2,
    },
    // 如果是主席台，则容纳的座位数量
    holdSeatNum: {
      type: Number,
      default: 0,
    },
    reqType: {
      type: Number,
      default: 1,
    }, // 请求类型 可拖拽，固定状态2 不可放置状态3
  },
  computed: {

    dragClass() {
     return  this.reqType === 1 ? 'drag' : (this.reqType === 2?'undrag':'dragUnset');
    },

    divWidth() {
      return this.facilityWidth;
    },
    divHeight() {
      return this.facilityHeight;
    },

    facilityBlockArray() {
      return this.renderFacility();
    },

    transformClass() {
      // 默认朝下
      switch (this.facilityOrientation) {
      case 'l':
        return 'divTransform90';
      case 'r':
        return 'divTransform270';
      case 'u':
        return 'divTransform180';
      case 'd':
        return '';
      default:
        return '';
      }
    },

    divTableClass(){
       return this.reqType === 1 ? 'divTableSelected' : (this.reqType === 2?'divTable':'divTableUnset');
    },

    rostrumSeatClass(){
        return this.reqType === 1 ? 'rostrum-seat-selected' : (this.reqType === 2?'rostrum-seat':'rostrum-seat-unset');
    },

    divRostrumClass(){
       return this.reqType === 1 ? 'divRostrum-selected' : (this.reqType === 2?'divRostrum':'divRostrum-unset');
    },
    divDoorClass(){
             return this.reqType === 1 ? 'divDoor-selected' : (this.reqType === 2?'divDoor':'divDoor-unset');
    },

    // 获取主席台编号 数组
    getArray() {
      let arr = Array.from(Array(this.holdSeatNum), (v, k) => k + 1);
      arr = this.getRostrumArr(arr).reverse();
      return arr;
    },
  },


  data() {
    return {
      emptyFlag: 0,
    };
  },
  methods: {

    // 主席台座位排序
    getRostrumArr(arr) {
      const arr2 = [];
      const n = arr.length;
      // 便于测试，由于空格不可见，所以调试时候可以使用*替代空格
      if (n % 2 == 0) {
        for (let i = n - 1; i > 0; i -= 2) {
          arr2.push(arr[i]);
        }
        // 1 2 3 4 5
        for (let i = 0; i < n; i += 2) {
				   arr2.push(arr[i]);
        }
      } else {
        for (let i = n - 1; i >= 0; i -= 2) {
			  arr2.push(arr[i]);
        }
        // 1 2 3 4 5
        for (let i = 1; i < n; i += 2) {
				 arr2.push(arr[i]);
        }
      }
      return arr2;
    },


  },
  mounted() {
    this.getArray;
  },

};
</script>

<style lang="scss" scoped>
  .drag{
      position: absolute;
      display:flex;
      justify-content:center;
      align-items:center;
      pointer-events: none;
      opacity: 1.0;
      border-radius: 2px;
      background-size: cover;
      background: rgba(13, 175, 156, 0.15);
  }
  .dragUnset{
       position: absolute;
      display:flex;
      justify-content:center;
      align-items:center;
      pointer-events: none;
      opacity: 1.0;
      border-radius: 2px;
      background-size: cover;
      background: rgba(253, 87, 81, 0.15);
  }
  .undrag{
    position:absolute;
    opacity: 1;
    display:flex;
    justify-content:center;
    align-items:center;
  }
  .divDoor{
     background: url('../../../../assets/image/web/seat/real-door.svg')
    center center no-repeat;
    background-size: 100% 100%;
  }
   .divDoor-selected{
     background: url('../../../../assets/image/web/seat/real-door-selected.svg')
    center center no-repeat;
    background-size: 100% 100%;
  }
  .divDoor-unset{
     background: url('../../../../assets/image/web/seat/real-door-unset.svg')
    center center no-repeat;
    background-size: 100% 100%;
  }
  .divTable{
     background: #E7D4C0;
     border-radius: 2px;
     width: 100%;
     height: 100%;
  }
  .divTableSelected{
     background: #0DAF9C;
     border-radius: 2px;
     width: 100%;
     height: 100%;
  }
  .divTableUnset{
     background: #FD5751;
     border-radius: 2px;
     width: 100%;
     height: 100%;
  }
  .divRostrumBox{
      display: flex;
      flex-direction: column ;
    }
  .divRostrumSeats{
      display: flex;
      flex-direction: row ;
  }
  .divRostrum{
    background: #9BC0DF;
    border-radius: 4px;
  }
  .divRostrum-selected{
    background: #6CCDC2;
    border-radius: 4px;
  }

  .divRostrum-unset{
    background: #FD5751;
    border-radius: 4px;
  }
 .rostrum-seat{
    border: white 5px solid;
    background: url('../../../../assets/image/web/seat/rostrum-seat-down.svg')
    center center no-repeat;
    background-size: 100% 100%;
    line-height: 42px;
  }
  .rostrum-seat-unset{
    border: white 5px solid;
    background: url('../../../../assets/image/web/seat/rostrum-seat-unset-down.svg')
    center center no-repeat;
    background-size: 100% 100%;
    line-height: 42px;
  }
  .rostrum-seat-selected{
    border: white 5px solid;
    background: url('../../../../assets/image/web/seat/rostrum-seat-selected-down.svg')
    center center no-repeat;
    background-size: 100% 100%;
    line-height: 42px;
  }

 .divMainScreen{
      display: flex;
      flex-direction: row ;
  }

   .divTransform90
  {
    transform:rotate(90deg);
    -ms-transform:rotate(90deg); /*IE 9*/
    -moz-transform:rotate(90deg);  /*Firefox*/
    -webkit-transform:rotate(90deg);  /*Safari和Chrome*/
    -o-transform:rotate(90deg);  /*Opera*/
  }
  .divTransform180
  {
    transform:rotate(180deg);
    -ms-transform:rotate(180deg);  /* IE 9*/
    -moz-transform:rotate(180deg);  /*Firefox*/
    -webkit-transform:rotate(180deg);  /*Safari和Chrome*/
    -o-transform:rotate(180deg);  /*Opera*/
  }
  .divTransform270
  {
    transform:rotate(270deg);
    -ms-transform:rotate(270deg); /* IE 9*/
    -moz-transform:rotate(270deg); /* Firefox*/
    -webkit-transform:rotate(270deg); /* Safari和Chrome*/
    -o-transform:rotate(270deg); /*Opera*/
  }
</style>
