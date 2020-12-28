<template>
<div :class="[dragClass, transformClass]" :style="{width:divWidth*(gridSize+doubleBorder)-doubleBorder+'px',
      height:divHeight*(gridSize+doubleBorder)-doubleBorder+'px'}">
     <!--会议桌-->
     <div v-if="facilityType==='table'" :class='divTableClass'
      :style="{width:divWidth*(gridSize+doubleBorder)-doubleBorder+'px',
      height:divHeight*(gridSize+doubleBorder)-doubleBorder+'px'}" >
    </div>
     <!--主席台-->
    <div v-else-if="facilityType === 'rostrum'"  :class='divRostrumBoxClass'
    :style="{width:divWidth*(gridSize+doubleBorder)-doubleBorder+'px',
         height:divHeight*(gridSize+doubleBorder)-doubleBorder+'px'}">
    <div class='divRostrumSeats' v-if="rostrumCount>0">
          <div
            v-for="(itme,index) in rostrumCount"
            :key="index"
            :class="rostrumSeatClass(index)"
            :style="rostrumSeatStyle"
            :value="getArray[index]"
            @click="onClickSeatArrage(seatMap['主'+getArray[index]])">
            <span
              v-if="!seatView"
              :title="getRostrumSeatUserNameBySeatIndex(index)"
            >{{ getRostrumSeatUserNameBySeatIndex(index, true) }}</span>
            <span v-else-if="!isPreview">主{{getArray[index]}}</span>
          </div>
        </div>
        <div :class='divRostrumClass' :style="{width:divWidth*(gridSize+doubleBorder)-doubleBorder+'px',
         height:divRostrumHeight*(gridSize+doubleBorder)-doubleBorder+'px'}">
        </div>
    </div>
     <!--门-->
    <div v-else-if="facilityType === 'door'"
        :style="{width:(gridSize+doubleBorder)*divWidth+'px',
        height:(gridSize+doubleBorder)*divHeight+'px'}" :class ='divDoorClass'>
    </div>
     <!--屏幕-->
    <div v-else-if="facilityType === 'mainScreen'" class='divMainScreen'
         :style="{width:divWidth*(gridSize+doubleBorder)-doubleBorder+'px',
      height:divHeight*(gridSize+doubleBorder)-doubleBorder+'px'}" >
       <img :src="reqType === 1?
       require('../../../../assets/image/web/seat/screen-selected-left.svg'):
      (reqType === 2?require('../../../../assets/image/web/seat/screen-left.svg'):
      require('../../../../assets/image/web/seat/screen-unset-left.svg'))"
       :style="{width:(gridSize+doubleBorder/2)+'px',
      height:(gridSize)+'px'}" class="bg-img">
       <img v-for="(itme,index) in divWidth-2" :key="index"
       :src="reqType === 1?
       require('../../../../assets/image/web/seat/screen-selected-middle.svg' ):
      (reqType === 2?require('../../../../assets/image/web/seat/screen-middle.svg'):
      require('../../../../assets/image/web/seat/screen-unset-middle.svg'))"
       :style="{width:(gridSize+doubleBorder)+'px',
      height:(gridSize)+'px'}" class="bg-img">
       <img  :src="reqType === 1?
       require('../../../../assets/image/web/seat/screen-selected-right.svg'):
      (reqType === 2?require('../../../../assets/image/web/seat/screen-right.svg'):
      require('../../../../assets/image/web/seat/screen-unset-right.svg'))"
       :style="{width:(gridSize+doubleBorder/2)+'px',
      height:(gridSize)+'px'}" class="bg-img">
    </div>
    <div v-else>
      请选择设施
    </div>
</div>
</template>

<script>
import { get, isEmpty, isFunction } from 'lodash';

export default {
  /** 画模型 */
  props: {
    gridSize: Number, // 单元格大小
    facilityWidth: Number, // 宽
    facilityHeight: Number, // 高
    facilityType: String, // 设施类型
    facilityOrientation: String, // 朝向
    rostrumHeight: Number, // 主席台高度
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
    // 请求类型 可拖拽，固定状态2 不可放置状态3
    reqType: {
      type: Number,
      default: 1,
    }, // 请求类型 可拖拽，固定状态2 不可放置状态3
    // 是否预览模式
    isPreview: {
      type: Boolean,
      default: false,
    },
    // 是否显示座位视图，默认true，若为false，则显示人员视图（用于主席台座位显示）
    seatView: {
      type: Boolean,
      default: true,
    },
    // 由座位号位key，座位信息为value（用于主席台座位排座）
    seatMap: {
      type: Object,
      default() {
        return {};
      },
    },
    // 分配主席台座位给人员
    handleSeatArrage: {
      type: Function,
    },
  },
  computed: {
    rostrumSeatStyle() {
      if (this.seatView) {
        return {
          width: `${this.gridSize}px`,
          height: `${this.gridSize}px`,
          margin: `${this.doubleBorder / 2}px`,
        };
      }
      return {
        width: '42px',
        height: '42px',
      };
    },
    dragClass() {
      if (this.reqType === 1) {
        return 'drag';
      } if (this.reqType === 2) {
        return 'undrag';
      }
      return 'dragUnset';
    },


    divRostrumHeight() {
      if (this.facilityType === 'rostrum') {
        return this.rostrumHeight > 8 ? 8 : this.rostrumHeight;
      }
      return this.rostrumHeight === '' ? 0 : this.rostrumHeight;
    },
    divWidth() {
      if (this.facilityType === 'mainScreen') {
        if (!this.facilityWidth) {
          return 3;
        }
        if (this.facilityWidth > 10) {
          return 10;
        }
        if (this.facilityWidth < 3) {
          return 3;
        }
        return this.facilityWidth;
      } if (this.facilityType === 'rostrum' || this.facilityType === 'table') {
        return this.facilityWidth > 12 ? 12 : this.facilityWidth;
      }
      return this.facilityWidth;
    },
    divHeight() {
      if (this.facilityType === 'table') {
        return this.facilityHeight > 12 ? 12 : this.facilityHeight;
      } if (this.facilityType === 'rostrum') {
        return this.facilityHeight > 9 ? 9 : this.facilityHeight;
      }
      return this.facilityHeight;
    },
    rostrumCount() {
      if (this.facilityType === 'rostrum') {
        return this.holdSeatNum > 12 ? 12 : this.holdSeatNum;
      }
      return this.holdSeatNum;
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
    divRostrumBoxClass() {
      if (this.isPreview) {
        return 'divRostrumBoxPre';
      }
      return 'divRostrumBox';
    },

    divTableClass() {
      if (this.reqType === 1) {
        return 'divTableSelected';
      } if (this.reqType === 2) {
        return 'divTable';
      }
      return 'divTableUnset';
    },

    rostrumSeatClass() {
      return (index) => {
        let rostrumSeatClass = '';
        if (this.seatView) { // 预览或者编辑模板时，座位视图
          const userName = get(this.seatMap[`主${this.getArray[index]}`], 'user_name', '');
          // 预览时，座位号已安排人员
          if (!isEmpty(this.seatMap) && !isEmpty(userName)) {
            rostrumSeatClass = 'rostrum-seat-selected rostrum-seat-span__selected';
          } else if (this.reqType === 1) {
            rostrumSeatClass = 'rostrum-seat-selected';
          } else if (this.reqType === 2) {
            rostrumSeatClass = 'rostrum-seat';
          } else {
            rostrumSeatClass = 'rostrum-seat-unset';
          }
        } else if (!isEmpty(this.seatMap)) { // 预览时，人员视图
          rostrumSeatClass = 'rostrum-seat__input';
        }
        return `rostrum-seat__common ${rostrumSeatClass}`;
      };
    },

    divRostrumClass() {
      if (this.reqType === 1) {
        return 'divRostrum-selected';
      } if (this.reqType === 2) {
        return 'divRostrum';
      }
      return 'divRostrum-unset';
    },
    divDoorClass() {
      if (this.reqType === 1) {
        return 'divDoor-selected';
      } if (this.reqType === 2) {
        return 'divDoor';
      }
      return 'divDoor-unset';
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
      if (n % 2 === 0) {
        for (let i = n - 1; i > 0; i -= 2) {
          arr2.push(arr[i]);
        }
        for (let i = 0; i < n; i += 2) {
          arr2.push(arr[i]);
        }
      } else {
        for (let i = n - 1; i >= 0; i -= 2) {
          arr2.push(arr[i]);
        }
        for (let i = 1; i < n; i += 2) {
          arr2.push(arr[i]);
        }
      }
      return arr2;
    },

    // 根据主席台座位数字下表，获取主席台座位人员名称
    getRostrumSeatUserNameBySeatIndex(index, format) {
      if (index < 0) return '';
      const name = get(this.seatMap[`主${this.getArray[index]}`], 'user_name', '');
      return format && name.length > 5 ? `${name.substr(0, 5)}…` : name;
    },

    // 点击座位进行分配的回调
    onClickSeatArrage(val) {
      if (
        !this.seatView
        && this.handleSeatArrage
        && isFunction(this.handleSeatArrage)
      ) {
        this.handleSeatArrage(val);
      }
    },
  },

};
</script>

<style lang="scss" scoped>
  .m5 {
    margin: 5px;
  }
  .drag{
      position: absolute;
      display:flex;
      justify-content:center;
      align-items:center;
      pointer-events: none;
      opacity: 0.6;
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
      opacity:0.6;
      border-radius: 2px;
      background-size: cover;
      background: rgba(253, 87, 81, 0.15);
  }
  .undrag{
    position:absolute;
    display:flex;
    justify-content:center;
    align-items:center;
    background-color: #FCFCFC;
  }
  .divDoor{
     background: url('../../../../assets/image/web/seat/real-door.svg')
    center center no-repeat;
    background-size:  100%;
    background-color:#FCFCFC;
  }
   .divDoor-selected{
     background: url('../../../../assets/image/web/seat/real-door-selected.svg')
    center center no-repeat;
    background-size:  100%;
  }
  .divDoor-unset{
     background: url('../../../../assets/image/web/seat/real-door-unset.svg')
    center center no-repeat;
    background-size: 100%;
  }
  .divTable{
     background: #E7D4C0;
     border-radius: 2px;
     width: 100%;
     height: 100%;
     opacity: 1;
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
     justify-content:center;
     align-items:center;
    }
   .divRostrumBoxPre{
      display: flex;
      flex-direction: column ;
       justify-content:center;
          align-items:center;
      border: 1px dashed #0DAF9C;
      border-radius: 4px;
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
    background: url('../../../../assets/image/web/seat/rostrum-seat-down.svg')
    center center no-repeat;
    background-size: 100% 100%;
  }
  .rostrum-seat-unset{
    background: url('../../../../assets/image/web/seat/rostrum-seat-unset-down.svg')
    center center no-repeat;
    background-size: 100% 100%;
  }
  .rostrum-seat-selected{
    background: url('../../../../assets/image/web/seat/rostrum-seat-selected-down.svg')
    center center no-repeat;
    background-size: 100% 100%;

  }
  .rostrum-seat-span__selected {
    color:#0DAF9C;
  }
  .rostrum-seat__common {
    width: 32px;
    height: 32px;
    display: flex;
    align-items: center;
    justify-content: center;
    span {
      font-size: 12px;
      text-align: center;
    }
  }
  .rostrum-seat__input {
    width: 42px;
    height: 42px;
    span {
      width: 37px;
      height: 28px;
      border-radius: 4px;
      word-break: break-all;
      border: 1px solid #333e5e;
      display: flex;
      align-items: center;
      justify-content: center;
    }
  }
 .divMainScreen{
      display: flex;
      flex-direction: row ;
       align-items: center;
      justify-content: center;
  }
.divTransform90
  {
    transform:rotate(90deg);
    -ms-transform:rotate(90deg); /*IE 9*/
    -moz-transform:rotate(90deg);  /*Firefox*/
    -webkit-transform:rotate(90deg);  /*Safari和Chrome*/
    -o-transform:rotate(90deg);  /*Opera*/
    span{
      transform:rotate(270deg);
        -ms-transform:rotate(270deg); /*IE 9*/
        -moz-transform:rotate(270deg);  /*Firefox*/
        -webkit-transform:rotate(270deg);  /*Safari和Chrome*/
        -o-transform:rotate(270deg);  /*Opera*/
        }
  }
  .divTransform180
  {
    transform:rotate(180deg);
    -ms-transform:rotate(180deg);  /* IE 9*/
    -moz-transform:rotate(180deg);  /*Firefox*/
    -webkit-transform:rotate(180deg);  /*Safari和Chrome*/
    -o-transform:rotate(180deg);  /*Opera*/
    span{
     transform:rotate(180deg);
    -ms-transform:rotate(180deg);  /* IE 9*/
    -moz-transform:rotate(180deg);  /*Firefox*/
    -webkit-transform:rotate(180deg);  /*Safari和Chrome*/
    -o-transform:rotate(180deg);  /*Opera*/
    }

  }
  .divTransform270
  {
    transform:rotate(270deg);
    -ms-transform:rotate(270deg); /* IE 9*/
    -moz-transform:rotate(270deg); /* Firefox*/
    -webkit-transform:rotate(270deg); /* Safari和Chrome*/
    -o-transform:rotate(270deg); /*Opera*/
    span{
    transform:rotate(90deg);
    -ms-transform:rotate(90deg); /* IE 9*/
    -moz-transform:rotate(90deg); /* Firefox*/
    -webkit-transform:rotate(90deg); /* Safari和Chrome*/
    -o-transform:rotate(90deg); /*Opera*/
    }
  }
</style>
