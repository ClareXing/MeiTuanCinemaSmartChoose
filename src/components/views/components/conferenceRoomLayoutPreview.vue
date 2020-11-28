<template>
 <div class="pr seatLayout" >
          <div class="seat-wrapper" ref="seatWrapperRef">
              <div class="inner-seat-wrapper"  ref="innerSeatWrapperRef">
                <!--模型布局-->
                <div v-for="(row,rowIndex) in seatTemplateData.layoutArray" :key="rowIndex"
                    style="display: flex;">
                  <div v-for="(item,colIndex) in row" :key="colIndex"
                    :class="['square-noneblock',seatStyle(seatTemplateData.layoutArray[rowIndex][colIndex].type,
                    seatTemplateData.layoutArray[rowIndex][colIndex].orientation)]"
                    :style="{width:gridSize+'px',height:gridSize+'px'}"  :row="rowIndex" :column="colIndex">
                  </div>
                </div>
              </div>
          </div>
          <!--设施-->
         <facility  v-for="(facility,index) in seatTemplateData.facilities" :key="index+'20000'" style="z-index:2000"
              :class="enableEventclass"
               :doubleBorder="10"
               :gridSize="gridSize"
               :facilityHeight="facility.facilityHeight"
               :facilityWidth="facility.facilityWidth"
               :facilityOrientation="facility.facilityOrientation"
               :facilityType="facility.facilityType"
               :holdSeatNum="facility.holdSeatNum"
               :style="{
                  top:(gridSize+doubleBorder)*facility.topIndex+diffLeftTop(facility.facilityOrientation,
                  facility.facilityWidth,facility.facilityHeight,gridSize+doubleBorder)+'px',
                  left:(gridSize+doubleBorder)*facility.leftIndex-diffLeftTop(facility.facilityOrientation,
                  facility.facilityWidth,facility.facilityHeight,gridSize+doubleBorder)+'px'}"
                >
               </facility>
           <!--围墙  -->
          <wall :borderWidth="gridSize" style="z-index:2000"
                :wallHeight="seatTemplateData.wallHeight"
                :wallWidth="seatTemplateData.wallWidth"
                :startLeftIndex="seatTemplateData.wallLeft"
                :startTopIndex="seatTemplateData.wallTop"
                :doorArea="seatTemplateData.doorArea"
                :style="{top:seatTemplateData.wallTop*(gridSize+doubleBorder)+gridSize+'px',
                     left:seatTemplateData.wallLeft*(gridSize+doubleBorder)+gridSize+'px'}"
               >
          </wall>
          <div class="tc mb0_item pb34 pt50" style="background: #F5F6FB;margin-bottom:0;">
                  <!-- <el-button type="primary" class="w100" plain @click="cancel">取消</el-button>
                  <el-button type="primary" class="w100" @click="saveAndQuit">保存并退出</el-button> -->
                </div>
  </div>

</template>
<script>
import $ from 'jquery';
import wall from './component/wall.vue';
import seats from './component/seats.vue';
import facility from './component/facility.vue';

export default {
  components: {
    wall,
    seats,
    facility,
  },
  props: {
    seatTemplateData: {}, // 单元格大小
    gridSize: {
      type: Number,
      default: 32,
    }, // 宽
    doubleBorder: {
      type: Number,
      default: 10,
    }, // 宽

  },
  // 计算属性 围墙的位置
  computed: {

    enableEventclass() {
      return this.enableEvent ? '' : 'unenable-events';
    },

    // 设施朝向左或者右 计算旋转后的top，left
    diffLeftTop() {
      return (orientation, width, height, unit) => {
        if (orientation === 'l' || orientation === 'r') {
          return (width - height) / 2 * unit;
        }
        return 0;
      };
    },

    transformClass() {
      // 默认朝下
      return (orientation) => {
        switch (orientation) {
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
      };
    },

    seatStyle() {
      return (type, orientation) => {
        if (type === 1) {
          switch (orientation) {
          case 'l':
            return 'normal-seat-left';
          case 'r':
            return 'normal-seat-right';
          case 'u':
            return 'normal-seat-up';
          case 'd':
            return 'normal-seat-down';
          default:
            return 'normal-seat-down';
          }
        } else if (type === 2) {
          switch (orientation) {
          case 'l':
            return 'senior-seat-left';
          case 'r':
            return 'senior-seat-right';
          case 'u':
            return 'senior-seat-up ';
          case 'd':
            return 'senior-seat-down';
          default:
            return 'senior-seat-down';
          }
        } else return '';
      };
    },
  },

  data() {
    return {

    };
  },

  methods: {

    // 初始座位数组
    initSeatArray() {

    },
  },

  mounted() {
    const oldArray = this.seatTemplateData.layoutArray;
  },

};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style scoped lang="scss">

   /* 编辑操作时的 背景框 */
  .mask {
    width: 100%;
    height: 100%;
    opacity: 0.6;
     position: fixed;
    top: 0;
    left: 0;
    background: grey;
    z-index: 998;
  }
  .seatLayout{
    flex: 1;
    display: flex;
    flex-direction: column;
    height: 100%;
    overflow: hidden;
  }

  .seat-wrapper{
    margin: 0 auto;
    position:relative;
    overflow: auto;
    background: #FCFCFC;
    border-radius: 2px;
    flex-grow:1;
    margin-bottom: 40px;
  }
  /*画布区域大小*/
  .inner-seat-wrapper{
    width:1000px;
    margin: 15px 15px ;
    top:10px;
    position: relative;
    bottom:0px;
    box-sizing: content-box;
    left:10px;
    z-index: 1000;
    background: #FCFCFC;
    border-radius: 2px;
    border: 0px solid #E4E8EB;

  }
  /*画布中小正方形 */
  .square-block{
    /* float:left; */
    border-radius: 2px;
    border: white 5px solid;
    background: #F2F4F5;
  }
 .square-noneblock{
    border: white 5px solid;
    background: white;
    z-index: 2000;
 }

.inner-seat{
    width:100%;
    height:100%;
    cursor: pointer;
  }
 .normal-seat-left{
     background: url('../../../assets/image/web/seat/normal-seat-left.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .normal-seat-right{
    background: url('../../../assets/image/web/seat/normal-seat-right.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
   .normal-seat-up{
   background: url('../../../assets/image/web/seat/normal-seat-up.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .normal-seat-down{
    background: url('../../../assets/image/web/seat/normal-seat-down.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
   .senior-seat-left{
     background: url('../../../assets/image/web/seat/senior-seat-left.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .senior-seat-right{
    background: url('../../../assets/image/web/seat/senior-seat-right.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
  .senior-seat-down{
     background: url('../../../assets/image/web/seat/senior-seat-down.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
     .senior-seat-up{
     background: url('../../../assets/image/web/seat/senior-seat-up.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }

  .senior-seat{
    background-size: 100% 100%;
  }
  .add-seat{
    pointer-events: none;
  }
  .edit-form {
    // pointer-events: none;
    position: absolute;
  }
  .div-facility{
    position:absolute;
    opacity: 1;
    display:flex;
    justify-content:center;
    align-items:center;
    z-index: 2000;
  }
  .unenable-events{
      pointer-events: none;
  }

  .grid-content {
    width: 132px;
    height: 132px;
    border-radius: 2px;
    border: 1px solid #D2E0F3;
     // 居中 （flex）
      // box-sizing:border-box;
    display: grid;
    vertical-align: middle;
    text-align: center;
    img {
       margin-left: 43px;
       margin-top: 25px;
       display: block;
    }
    &:hover{
      background: #F3FFFE;
      #table{
       content: url('../../../assets/image/web/seat/table-hover.svg');
      }
      #mainScreen{
       content: url('../../../assets/image/web/seat/mainScreen-hover.svg');
      }
      #rostrum{
        content: url('../../../assets/image/web/seat/rostrum-hover.svg');
      }
      #door{
        content: url('../../../assets/image/web/seat/door-hover.svg');
      }
    }
  }
  .dialog-model{
    height: 252px;
    background: #FFFFFF;

  }
 .dialog-model-detail{
    height: 390px;
  }
  .div-preview{
    display:flex;
    justify-content:center;
    align-items:center;
    width: 197px;
    height: 198px;
    background: #F4FFFE;
    opacity: 0.42;
    border: 1px solid #86D7CD;
  }
 .rostrum-seat{
    border: white 5px solid;
    background: url('../../../assets/image/web/seat/rostrum-seat-down.svg')
    center center no-repeat;
    background-size: 100% 100%;
    line-height: 42px;
  }
    .divDoor{
     background: url('../../../assets/image/web/seat/real-door.svg')
    center center no-repeat;
    background-size: 100% 100%;
  }
  .divTable{
     background: #E7D4C0;
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
      border-radius: 2px;
  }
 .divMainScreen{
     font-size: 0%;
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
  .tipSpan{
      font-size: 10px;
      font-family: PingFangSC-Regular, PingFang SC;
      font-weight: 400;
      color: #B2C7E3;
      line-height: 14px;
  }

</style>
