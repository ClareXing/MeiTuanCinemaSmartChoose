<template>
  <div class="pr seat-layout__wrapper" :style="seatLayoutWrapperStlye">
    <div class="seat-layout__inner" :style="wallStyle">
      <!--模型布局-->
      <template v-if="isvisible">
        <div
          v-for="(row, rowIndex) in seatTemplateData.layoutArray"
          :key="rowIndex"
          class="seat-wrapper__item"
          v-show="rowIndex >= realPos.minTop && rowIndex <= realPos.maxButtom"
        >
          <div
            v-for="(item, colIndex) in row"
            :key="colIndex"
            :row="rowIndex"
            :column="colIndex"
            :class="[
              !seatView && (item.type === 1 || item.type === 2)
                ? 'name-block'
                : 'square-noneblock',
            ]"
            v-show="colIndex >= realPos.minLeft && colIndex <= realPos.maxRight"
          >
            <div
              v-if="seatView"
              :class="seatStyle(item.type, item.orientation, item.id)"
            >
              <span v-if="item.type === 1 || item.type === 2">{{
                item.id
              }}</span>
            </div>
            <div
              class="name-block__empty"
              v-if="!seatView && (item.type === 1 || item.type === 2)"
              @click="onClickSeatArrage(seatMap[item.id])"
            >
              <span :title="getSeatUserNameBySeatNo(item.id)">{{
                getSeatUserNameBySeatNo(item.id, true)
              }}</span>
            </div>
          </div>
        </div>
      </template>
    </div>
    <!--设施-->
    <div
      v-for="(facility, index) in seatTemplateData.facilities"
      :key="index + '20000'"
    >
      <facility
        :doubleBorder="doubleBorder"
        :gridSize="gridSize"
        :facilityHeight="facility.facilityHeight"
        :facilityWidth="facility.facilityWidth"
        :facilityOrientation="facility.facilityOrientation"
        :facilityType="facility.facilityType"
        :holdSeatNum="facility.holdSeatNum"
        :rostrumHeight="facility.rostrumHeight"
        :reqType="2"
        :style="facilityStyle(facility)"
        :seatView="seatView"
        :seatMap="seatMap"
        :handleSeatArrage="handleSeatArrage"
      >
      </facility>
    </div>
    <!--围墙  -->
    <wall
      :borderWidth="10"
      :wallHeight="seatTemplateData.wallHeight"
      :wallWidth="seatTemplateData.wallWidth"
      :startLeftIndex="seatTemplateData.wallLeft"
      :startTopIndex="seatTemplateData.wallTop"
      :doorArea="seatTemplateData.doorArea"
      :style="wallStyle"
      :gridSize="gridSize"
    >
    </wall>
  </div>
</template>
<script>
import {
  get, isEmpty, isFunction, isString,
} from 'lodash';
import wall from './component/wall.vue';
import facility from './component/facility.vue';

export default {
  components: {
    wall,
    facility,
  },
  props: {
    width: {
      type: String,
      default: '1008px',
    },
    height: {
      type: String,
      default: '840px',
    },
    // 座位布局模板
    seatTemplateData: {
      type: Object,
      default() {
        return {
          layoutArray: [],
        };
      },
    },
    // 单元格大小
    gridSize: {
      type: Number,
      default: 32,
    }, // 宽
    doubleBorder: {
      type: Number,
      default: 10,
    }, // 宽
    // 是否显示座位视图，默认true，若为false，则显示人员视图
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
    // 分配座位给人员
    handleSeatArrage: {
      type: Function,
    },
    // 切换布局视图时，设置其样式
    setSwitchSeatLayoutItemStyle: {
      type: Function,
    },
  },
  // 计算属性 围墙的位置
  computed: {
    // 整个布局是否可见
    isvisible() {
      return (
        !isEmpty(this.seatTemplateData)
        && !isEmpty(this.seatTemplateData.layoutArray)
      );
    },

    // 计算去除围墙之外的空白格后，上下左右的极限位置
    realPos() {
      const {
        wallTop, wallLeft, wallWidth, wallHeight,
      } = this.seatTemplateData;
      return {
        minTop: wallTop || 0,
        minLeft: wallLeft || 0,
        maxButtom: parseInt(wallTop, 10) + parseInt(wallHeight, 10) - 1,
        maxRight: parseInt(wallLeft, 10) + parseInt(wallWidth, 10) - 1,
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
      return (type, orientation, seat_no) => {
        // 预览座位视图下，座位是否已被选中
        const isSeatViewSelected = this.seatMap[seat_no]
          && !isEmpty(get(this.seatMap[seat_no], 'user_name'));

        let style = '';
        if (type === 1) {
          switch (orientation) {
          case 'l':
            style = isSeatViewSelected ? 'normal-seat-selected-left' : 'normal-seat-left';
            break;
          case 'r':
            style = isSeatViewSelected ? 'normal-seat-selected-right' : 'normal-seat-right';
            break;
          case 'u':
            style = isSeatViewSelected ? 'normal-seat-selected-up' : 'normal-seat-up';
            break;
          case 'd':
            style = isSeatViewSelected ? 'normal-seat-selected-down' : 'normal-seat-down';
            break;
          default:
            style = isSeatViewSelected ? 'normal-seat-selected-down' : 'normal-seat-down';
            break;
          }
        } else if (type === 2) {
          switch (orientation) {
          case 'l':
            style = isSeatViewSelected ? 'senior-seat-selected-left' : 'senior-seat-left';
            break;
          case 'r':
            style = isSeatViewSelected ? 'senior-seat-selected-right' : 'senior-seat-right';
            break;
          case 'u':
            style = isSeatViewSelected ? 'senior-seat-selected-up' : 'senior-seat-up';
            break;
          case 'd':
            style = isSeatViewSelected ? 'senior-seat-selected-down' : 'senior-seat-down';
            break;
          default:
            style = isSeatViewSelected ? 'senior-seat-selected-down' : 'senior-seat-down';
            break;
          }
        }

        if (isSeatViewSelected) style += ' normal-seat__selected';
        return `normal-seat__common ${style}`;
      };
    },

    facilityStyle() {
      return (val) => {
        const {
          topIndex,
          leftIndex,
          facilityWidth,
          facilityHeight,
          facilityOrientation,
        } = val;
        const sideLen = this.gridSize + this.doubleBorder;
        const diff = this.diffLeftTop(
          facilityOrientation,
          facilityWidth,
          facilityHeight,
          sideLen,
        );
        const { minTop, minLeft } = this.realPos;
        const top = sideLen * (topIndex - minTop) + diff;
        const left = sideLen * (leftIndex - minLeft) - diff;
        return {
          top: `${top}px`,
          left: `${left}px`,
        };
      };
    },

    wallStyle() {
      const {
        wallWidth,
        wallHeight,
      } = this.seatTemplateData;
      const sideLen = this.gridSize + this.doubleBorder;
      return {
        top: '0px',
        left: '0px',
        width: `${wallWidth * sideLen}px`,
        height: `${wallHeight * sideLen}px`,
      };
    },

    // 计算布局上下左右偏移量，使其居中显示
    seatLayoutWrapperStlye() {
      const cutPX = (val) => {
        if (!isString(val)) return val;
        return parseInt(val.substr(0, val.length - 2), 10);
      };
      const { width, height } = this.wallStyle;
      const wallWidth = cutPX(width);
      const wallHeight = cutPX(height);
      const wrapperWidth = cutPX(this.width);
      const wrapperHeight = cutPX(this.height);
      let left = 0;
      let right = 0;
      let top = 0;
      let bottom = 0;
      if (wallWidth < wrapperWidth) {
        left = Math.round((wrapperWidth - wallWidth) / 2, 2);
        right = wrapperWidth - wallWidth - left;
      }
      if (wallHeight < wrapperHeight) {
        top = Math.round((wrapperHeight - wallHeight) / 2, 2);
        bottom = wrapperHeight - wallHeight - top;
      }
      return {
        margin: `${top}px ${right}px ${bottom}px ${left}px`,
      };
    },
  },

  mounted() {
    const _this = this;
    if (
      _this.setSwitchSeatLayoutItemStyle
        && isFunction(_this.setSwitchSeatLayoutItemStyle)
    ) {
      _this.setSwitchSeatLayoutItemStyle();
    }
  },

  data() {
    return {};
  },

  methods: {
    // 设施朝向左或者右 计算旋转后的top，left
    diffLeftTop(orientation, width, height, unit) {
      if (orientation === 'l' || orientation === 'r') {
        return ((width - height) / 2) * unit;
      }
      return 0;
    },

    getSeatUserNameBySeatNo(val, format) {
      if (isEmpty(val) || isEmpty(this.seatMap)) return '';
      const name = get(this.seatMap, `${val}.user_name`, '');
      return format && name.length > 5 ? `${name.substr(0, 5)}…` : name;
    },

    // 点击座位进行分配的回调
    onClickSeatArrage(val) {
      if (this.handleSeatArrage && isFunction(this.handleSeatArrage)) {
        this.handleSeatArrage(val);
      }
    },
  },
};
</script>
<style scoped lang="scss">
.seat-layout__wrapper {
  .seat-layout__inner {
    width: 1008px;
    height: 840px;
    background-color: #fff;
    .seat-wrapper__item {
      display: flex;
      .name-block {
        width: 42px;
        height: 42px;
        display: flex;
        align-items: center;
        justify-content: center;
        .name-block__empty {
          width: 37px;
          height: 28px;
          border-radius: 4px;
          border: 1px solid #333e5e;
          span {
            width: 100%;
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 12px;
            text-align: center;
            word-break: break-all;
          }
        }
      }
      .square-noneblock {
        width: 32px;
        height: 32px;
        margin: 5px;
        background: white;
        display: flex;
        align-items: center;
        justify-content: center;
        .normal-seat__selected {
          color: #0daf9c;
        }
        .normal-seat__common {
          width: 100%;
          height: 100%;
          font-size: 12px;
          display: flex;
          justify-content: center;
          align-items: center;
        }
      }
    }
  }
}
</style>
<style lang="less" scoped>
.seat-img-common(@imgName) {
  background: url('../../../assets/image/web/seat/@{imgName}.svg');
  background-size: 100% 100%;
}
.normal-seat-left {
  .seat-img-common('normal-seat-left');
}
.normal-seat-right {
  .seat-img-common('normal-seat-right');
}
.normal-seat-up {
  .seat-img-common('normal-seat-up');
}
.normal-seat-down {
  .seat-img-common('normal-seat-down');
}
.normal-seat-selected-left {
  .seat-img-common('normal-seat-selected-left');
}
.normal-seat-selected-right {
  .seat-img-common('normal-seat-selected-right');
}
.normal-seat-selected-up {
  .seat-img-common('normal-seat-selected-up');
}
.normal-seat-selected-down {
  .seat-img-common('normal-seat-selected-down');
}
.senior-seat-left {
  .seat-img-common('senior-seat-left');
}
.senior-seat-right {
  .seat-img-common('senior-seat-right');
}
.senior-seat-down {
  .seat-img-common('senior-seat-down');
}
.senior-seat-up {
  .seat-img-common('senior-seat-up');
}
.senior-seat-selected-left {
  .seat-img-common('senior-seat-selected-left');
}
.senior-seat-selected-right {
  .seat-img-common('senior-seat-selected-right');
}
.senior-seat-selected-down {
  .seat-img-common('senior-seat-selected-down');
}
.senior-seat-selected-up {
  .seat-img-common('senior-seat-selected-up');
}
</style>
