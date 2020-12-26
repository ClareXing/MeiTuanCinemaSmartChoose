<template>
<div class="pr seatLayout" ref='seatLayoutRef'>

    <div class="clearfix" style="padding-left: 20px ;margin-bottom:20px">
    <el-form :rules="rules" ref='templateform' :model="seatTemplate">
        <el-form-item prop="name" style="margin-bottom: 0">
        <el-input v-if="nameEdit" class="vue-input"
          ref="nameInput"
          @blur="nameEdit = false"
          maxlength="20"
          size="medium"
          placeholder="请输入布局名称"
          v-model.trim="seatTemplate.name">
        </el-input>
         <div v-else style="height: 36px;display: flex;align-items: center;">{{seatTemplate.name}}
          <span class="el-icon-edit-outline name-edit-icon" @click="editName"></span>
          </div>
        </el-form-item>
    </el-form>
        <hr style="border:0;height:2px;background: #F5F6FB;margin: 0" />
     </div>
    <div class="btn-wrapper">
      <div class="left-wrapper">
        <el-button type="primary"  @click="selModel">添加模型</el-button>
        <el-button type="primary"  @click="addFacility">添加设施</el-button>
        <el-button class="addSeat"
        @click="addSeat" :type="!enableAddSeat?'primary' : ''">添加座位</el-button>
        <el-button class="clear-all" type="text" icon="el-icon-delete"
        :disabled="seatTemplate.wallWidth===0"
        @click="clearAll">清空全部</el-button>
      </div>
      <div class='div-icon'>
        <div></div><span>主席台</span>
        <div></div><span>会议桌</span>
        <div></div><span>主屏幕</span>
      </div>
    </div>
    <div ref='mainSeatAreaRef' class="mainSeatArea">
     <div class="seat-wrapper" ref="seatWrapperRef">
        <div class="inner-seat-wrapper unselect "  @mousedown="mousedown" @mouseup="mouseup"
        @mousemove.stop='mousemove'
        ref="innerSeatWrapperRef" draggable="false">
          <!--模型布局-->
          <p class="v-selected-area" v-show="isMouseDown" ref="selectedArea"></p>
          <div v-for="(row,rowIndex) in seatTemplate.layoutArray" :key="rowIndex"
          style="display: flex;">
            <div v-for="(item,colIndex) in row" :key="colIndex"
         :class="['square-block',seatStyle(item.type,item.orientation,
          typeof(item.selected)==='undefined'||!item.selected?false:true)]"
            :style="{width:gridSize+'px',height:gridSize+'px'}" :row="rowIndex" :column="colIndex">
              <div class ="block-area"
              :style="{width:(gridSize+doubleBorder)+'px',height:(gridSize+doubleBorder)+'px',
             top:rowIndex*(gridSize+doubleBorder)+'px',left:colIndex*(gridSize+doubleBorder)+'px'}"
              :row="rowIndex" :column="colIndex">
              </div>
            </div>
          </div>
          <!--设施-->
          <div  :class="enableEventclass" @click="editFacility(facility)"
          v-for="(facility,index) in seatTemplate.facilities" :key="index+'20000'" >
            <facility
                :doubleBorder="10"
                :gridSize="gridSize"
                :facilityHeight="facility.facilityHeight"
                :facilityWidth="facility.facilityWidth"
                :facilityOrientation="facility.facilityOrientation"
                :facilityType="facility.facilityType"
                :holdSeatNum="facility.holdSeatNum"
                :rostrumHeight="facility.rostrumHeight"
                :style="{
                  top:(gridSize+doubleBorder)*facility.topIndex
                  +diffLeftTop(facility.facilityOrientation,
                  facility.facilityWidth,facility.facilityHeight,gridSize+doubleBorder)+'px',
                  left:(gridSize+doubleBorder)*facility.leftIndex
                  -diffLeftTop(facility.facilityOrientation,
                  facility.facilityWidth,facility.facilityHeight,gridSize+doubleBorder)+'px'}"
                  :reqType="facility.selected?1:2">
            </facility>
          </div>
          <!--弹出座位编辑框-->
          <el-popover placement="bottom"
                  width="314" trigger="manual"
                  :title="editInfo.type"
                  v-model="showMultiSeatsPop"
                  >
              <hr style="border:0;height:1px;background: #E4E8EE;" />
                <eidtform
                  :editInfo="editInfo"
                  :width="290"
                  :height="160"
                  :seatOrientation.sync="seatOrientation"
                  :seatType.sync="seatType"
                  :refreshValue = "new Date().getTime()"
                  @deleteItems="deleteItems"
                  @okEdit="okEdit"
                  @cancelEdit="cancelEdit"
                  >
                </eidtform>
                <div  slot="reference" class='popDiv' ref='popDiv'></div>
           </el-popover>
          <!--设施弹出pop 框-->
          <el-popover placement="bottom"
           width="314" trigger="manual"
           :title="editInfo.type"
           v-model="showFacilityPop"
           >
             <hr style="border:0;height:1px;background: #E4E8EE;" />
              <eidtform
                :editInfo="editInfo"
                :width="290"
                :height="100"
                 @deleteItems="deleteItems"
                 @cancelEdit="cancelEdit"
                 @okEdit="okEdit"
                >
              </eidtform>
                <div slot="reference" class='popDiv' ref='popFacilityEditDiv'></div>
           </el-popover>
           <!--围墙  -->
          <wall :borderWidth="10"  ref="wallRef"
                :wallHeight="seatTemplate.wallHeight"
                :wallWidth="seatTemplate.wallWidth"
                :startLeftIndex="seatTemplate.wallLeft"
                :startTopIndex="seatTemplate.wallTop"
                :doorArea="seatTemplate.doorArea"
                :style="{top:seatTemplate.wallTop*(gridSize+doubleBorder)+'px',
                         left:seatTemplate.wallLeft*(gridSize+doubleBorder)+'px'}"
                :gridSize="gridSize"
                @addDoor="addDoor"
                v-if='(!editState||facilityInfo.facilityType==="door")&&seatTemplate.wallWidth>0'>
          </wall>

         <!--添加座位时的图片-->
          <div v-show="enableAddSeat" ref="addSeatDivRef" id='addSeatDivRef'
          :class="forbidden?'add-seat-unset':'add-seat'">
            <img :src="forbidden?
            require('../../../assets/image/web/seat/normal-seat-unset-up.svg'):
            require('../../../assets/image/web/seat/normal-seat-selected-up.svg')"/>
          </div>

          <!--可移动状态座位模型鼠标点击定位时消失-->
          <seats @initSeatModelArray='initSeatModelArray'
               :type="seatModelInfo.seatModelType+'座位类型'"  ref="seatsRef"
                :gridSize="parseInt(gridSize)"
                :circleNum="parseInt(seatModelInfo.circleNum)"
                :seatModelArrayHeight="parseInt(seatModelArrayHeight)"
                :seatModelArrayWidth="parseInt(seatModelArrayWidth)"
                :modelType="seatModelInfo.seatModelType"
                :orientation="seatModelInfo.orientation" v-if="showDraggModel"
                :reqType="forbidden?3:1">
          </seats>
          <!--可移动设施时显示-->
          <facility :type="'设施'" ref="facilityRef"
                :gridSize="gridSize"
                :topIndex="facilityInfo.topIndex"
                :leftIndex="facilityInfo.leftIndex"
                :facilityWidth="facilityInfo.facilityWidth"
                :facilityHeight="facilityInfo.facilityHeight"
                :facilityType="facilityInfo.facilityType"
                :holdSeatNum="facilityInfo.holdSeatNum"
                :rostrumHeight="facilityInfo.rostrumHeight"
                :facilityOrientation="facilityInfo.facilityOrientation"
                v-if="showDraggFacility" :reqType="forbidden?3:1">
          </facility>
         </div>
     </div>

       <!--弹出的对话框区域-->
        <el-dialog title="添加模型" :visible.sync="dialogModelSelectVisible"
           width="694px" custom-class='dialog-model'
          :close-on-click-modal="false"  @close="clearValidates('seatModelFormRef')">
          <el-row :gutter="20">
            <el-col :span="6">
              <div class="grid-content"  @click="nextModel('uShape')">
               <img src="../../../assets/image/web/seat/uShape.svg" />
               <span >U型排布</span>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content" @click="nextModel('gyrationShape')">
                 <img src="../../../assets/image/web/seat/gyrationShape.svg"  />
                 <span >回型排布</span>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content" @click="nextModel('deskShape')">
              <img src="../../../assets/image/web/seat/deskShape.svg"  />
              <span >课桌式排布</span>
              </div>
          </el-col>
            <el-col :span="6">
              <div class="grid-content" @click="nextModel('customize')">
               <img src="../../../assets/image/web/seat/customize.svg"  />
               <span >自定义排布</span>
             </div>
            </el-col>
          </el-row>
        </el-dialog>

        <el-dialog title="编辑模型" :visible.sync="dialogModelInfoVisible"
        width="652px"  custom-class='dialog-model-detail'
        :close-on-click-modal="false" >
            <el-row>
               <el-col :span="16">
                  <el-form  label-position="left" :rules="seatModelRules"
                  ref='seatModelFormRef' :model="seatModelInfo">
                    <el-form-item label="U开口方向" label-width="105px"
                   v-show="seatModelInfo.seatModelType==='uShape'" >
                        <el-select v-model="seatModelInfo.orientation"
                        placeholder="请选择" style="width:275px">
                          <el-option
                            v-for="item in orientations"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value">
                          </el-option>
                        </el-select>
                    </el-form-item>
                  <el-row>
                    <el-col :span="12">
                        <el-form-item label-width="105px" prop="horizontalSeatNum"
                        :label="seatModelInfo.seatModelType==='uShape'?'U底部座位':'横向座位数'">
                          <el-input  v-model.number="seatModelInfo.horizontalSeatNum"
                          class="param-input"
                          ></el-input>
                           </el-form-item>
                    </el-col>
                    <el-col :span="12">
                        <el-form-item label-width="105px" prop="verticalSeatNum"
                        :label="this.seatModelInfo.seatModelType==='uShape'?'U侧边座位':'纵向座位数'">
                            <el-input v-model.number="seatModelInfo.verticalSeatNum"
                             class="param-input">
                            </el-input>
                        </el-form-item>
                    </el-col>
                 </el-row>
              <!--模型预览-->
              </el-form>
              </el-col>
              <el-col :span="8">
                  <div style="text-align:center;"  :class="isIE()?'div-preview-ie':''"
                  class="div-preview">
                      <seats  v-model="seatModelInfo"
                      :gridSize="8"
                      :circleNum="parseInt(seatModelInfo.circleNum)"
                      :seatModelArrayHeight="parseInt(seatModelArrayHeight)"
                      :seatModelArrayWidth="parseInt(seatModelArrayWidth)"
                      :modelType="seatModelInfo.seatModelType"
                      :orientation="seatModelInfo.orientation"
                      :doubleBorder="4"
                      :reqType="4">
                     </seats>
                  </div>
              </el-col>
            </el-row>
            <div slot="footer" class="dialog-footer">
              <el-button @click="upModel" type="primary" class="w80" plain>上一步</el-button>
              <el-button type="primary" @click="okModel" class='w80'>确 定</el-button>
            </div>
        </el-dialog>

        <el-dialog title="添加设施" :visible.sync="dialogFacilityVisible"
          width="694px" :close-on-click-modal="false" @close="clearValidates('facilityFormRef')"
          custom-class='dialog-model'>
          <el-row :gutter="20">
          <el-col :span="6">
            <div :class="containRostrum?'grid-content-disable':'grid-content'"
            @click="nextFacility('rostrum')" >
             <div id='rostrum'></div>
              <span>主席台</span><br>
              <span class='tipsSpan'>仅可添加一个</span>
            </div>
          </el-col>
          <el-col :span="6">
            <div :class="seatTemplate.wallWidth===0?'grid-content-disable':'grid-content'"
             @click="nextFacility('door')">
             <div id='door'></div>
                <span>入口</span>
                <br>
                <span class='tipsSpan'>围墙变化后需重新放置</span>
            </div>
         </el-col>
          <el-col :span="6">
          <div class="grid-content" @click="nextFacility('table')">
            <div id='table'></div>
             <span>会议桌</span>
          </div>
        </el-col>
          <el-col :span="6"><div class="grid-content" @click="nextFacility('mainScreen')">
             <div id='mainScreen'></div>
              <span>主屏幕</span>
           </div>
          </el-col>

        </el-row>
        </el-dialog>

        <el-dialog :title="editInfo.id===''?'添加设施':'编辑设施'"
        :visible.sync="dialogFacilityDetailVisible"  @close='cancelSelected'
        :close-on-click-modal="false"
         width="652px"  custom-class='dialog-model-detail'>
           <el-row>
            <el-col :span="16">
            <el-form  label-position="left" :rules="facilityRules"
             ref='facilityFormRef' :model="facilityInfo">
              <el-form-item label="设施朝向" v-show="facilityInfo.facilityType!=='table'"
              label-width="105px">
                <el-select v-model="facilityInfo.facilityOrientation"
                    placeholder="请选择" style="width:275px">
                  <el-option
                    v-for="item in orientations"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                  </el-option>
              </el-select>
              </el-form-item>
              <el-row>
                <el-col :span="12">
              <el-form-item label="设施长度" label-width="105px"
                v-if="facilityInfo.facilityType!=='door'" prop="facilityWidth">
                    <el-input   v-model.number="facilityInfo.facilityWidth"
                     @input="checkHoldSeats" class="param-input">

                    </el-input>
              </el-form-item>
                </el-col>
                <el-col :span="12">
                  <!--如果是主席台（设置单独主席台高度）-->
                  <el-form-item label="设施宽度" label-width="105px"
                  v-if="facilityInfo.facilityType==='rostrum'"  prop="rostrumHeight" >
                           <el-input   v-model.number="facilityInfo.rostrumHeight"
                          class="param-input" ></el-input>
                </el-form-item>
                <!--如果是会议桌-->
                  <el-form-item label="设施宽度" label-width="105px"
                  v-if="facilityInfo.facilityType==='table'"  prop="facilityHeight">
                  <el-input  v-model.number="facilityInfo.facilityHeight"
                  class="param-input"></el-input>
                </el-form-item>
                </el-col>
              </el-row>
            <el-form-item label="座位数量" label-width="105px"
               v-if="facilityInfo.facilityType==='rostrum'"   prop="holdSeatNum" >
                     <el-input   v-model.number="facilityInfo.holdSeatNum"
                    class="param-input"></el-input>
            </el-form-item>
          </el-form>
        </el-col>
          <el-col :span="8">
            <div  :class="isIE()?'div-preview-ie':''"
            class="div-preview" style="text-align:center;" >
              <facility class="child-pre"
                :gridSize="parseInt(14)"
                :facilityWidth="parseInt(facilityInfo.facilityWidth)"
                :facilityHeight="parseInt(facilityInfo.facilityHeight)"
                :facilityType="facilityInfo.facilityType"
                :holdSeatNum="parseInt(facilityInfo.holdSeatNum)"
                :facilityOrientation="facilityInfo.facilityOrientation"
                :rostrumHeight="facilityInfo.rostrumHeight"
                :reqType="2"
                :doubleBorder="2"
                :isPreview="true">
              </facility>
            </div>
          </el-col>
          </el-row>

        <div slot="footer" class="dialog-footer">
          <el-button @click="upFacility" v-if="editInfo.id===''"
          type="primary" class="w80" plain>上一步</el-button>
          <el-button type="primary" @click="okFacility" class='w80'>确 定</el-button>
        </div>
        </el-dialog>

       <!--编辑座位时候的遮罩层-->
      <div class="mask" @click="endEditState" v-if="editState"></div>
      <div class="tc mb0_item pb34 pt50" style="background: #F5F6FB;margin-bottom:0;">
      <el-button type="primary" class="w100" plain @click="cancel">取消</el-button>
      <el-button type="primary" class="w100" @click="saveAndQuit">保存并退出</el-button>
    </div>
    </div>
  </div>
</template>
<script>
import $ from 'jquery';
import wall from './component/wall.vue';
import seats from './component/seats.vue';
import facility from './component/facility.vue';
import eidtform from './component/eidtform.vue';

export default {
  components: {
    wall,
    seats,
    facility,
    eidtform,
  },
  // 监听属性
  watch: {
    // 计算围墙
    editState(val) {
      if (!val) {
        this.computeWall();
      }
    },
    // eslint-disable-next-line func-names
    'facilityInfo.rostrumHeight': function (val) {
      if (this.facilityInfo.facilityType !== 'rostrum') {
        return;
      }
      if (this.facilityInfo.holdSeatNum > 0) {
        this.facilityInfo.facilityHeight = val + 1;
      } else {
        this.facilityInfo.facilityHeight = val;
      }
    },
    // eslint-disable-next-line func-names
    'facilityInfo.holdSeatNum': function (val) {
      if (this.facilityInfo.facilityType !== 'rostrum') {
        return;
      }
      if (val > 0) {
        this.facilityInfo.facilityHeight = this.facilityInfo.rostrumHeight + 1;
      } else {
        this.facilityInfo.facilityHeight = this.facilityInfo.rostrumHeight;
      }
    },
    // 座位数小于等于主席台长度
    // 'facilityInfo.facilityWidth': function (val) {
    //   if (val < this.facilityInfo.holdSeatNum) {
    //     this.facilityInfo.holdSeatNum = val;
    //   }
    // },

  },
  // 计算属性 围墙的位置
  computed: {
    // 绑定 朝向 和门的方向
    orientations() {
      if (this.facilityInfo.facilityType === 'door' && this.dialogFacilityDetailVisible) {
        return [{
          value: 'u',
          label: '横向',
        }, {
          value: 'l',
          label: '纵向',
        }];
      }
      return [{
        value: 'u',
        label: '朝上',
      }, {
        value: 'd',
        label: '朝下',
      }, {
        value: 'l',
        label: '朝左',
      }, {
        value: 'r',
        label: '朝右',
      }];
    },
    // 根据座位数计算模型座位区域宽度
    seatModelArrayWidth() {
      // 如果类型是'uShape'
      if (this.seatModelInfo.seatModelType === 'uShape') {
        // 朝上或者朝下
        if (this.seatModelInfo.orientation === 'u' || this.seatModelInfo.orientation === 'd') {
          return this.seatModelInfo.horizontalSeatNum + 2;
        }// 朝左或者朝右
        return this.seatModelInfo.verticalSeatNum + 1;

        // 回型'gyrationShape
      } if (this.seatModelInfo.seatModelType === 'gyrationShape') {
        return this.seatModelInfo.horizontalSeatNum + 2;
      }
      return this.seatModelInfo.horizontalSeatNum;
    },
    // 根据座位数计算模型座位区域高度
    seatModelArrayHeight() {
      // 如果类型是'uShape'
      if (this.seatModelInfo.seatModelType === 'uShape') {
        // 朝上或者朝下
        if (this.seatModelInfo.orientation === 'u' || this.seatModelInfo.orientation === 'd') {
          return this.seatModelInfo.verticalSeatNum + 1;
        }// 朝左或者朝右
        return this.seatModelInfo.horizontalSeatNum + 2;
      } if (this.seatModelInfo.seatModelType === 'gyrationShape') { // 如果类型是 ,'gyrationShape
        return this.seatModelInfo.verticalSeatNum + 2;
      }
      return this.seatModelInfo.verticalSeatNum;
    },
    // 根据状态计算设施是否可以编辑
    enableEventclass() {
      return this.enableEvent ? '' : 'unenable-events';
    },

    // 是否编辑状态
    editState() {
      if (this.enableAddSeat || this.showDraggModel
      || this.showFacilityPop || this.showMultiSeatsPop
      || this.showDraggFacility || this.showEditForm
      || this.dialogFacilityDetailVisible) {
        return true;
      }
      return false;
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

    // 不同座位显示不同状态
    seatStyle() {
      return (type, orientation, selected) => {
        if (type === 1) {
          switch (orientation) {
          case 'l':
            return selected ? 'normal-seat-selected-left selectSeat' : 'normal-seat-left selectSeat';
          case 'r':
            return selected ? 'normal-seat-selected-right selectSeat' : 'normal-seat-right selectSeat';
          case 'u':
            return selected ? 'normal-seat-selected-up selectSeat' : 'normal-seat-up selectSeat';
          case 'd':
            return selected ? 'normal-seat-selected-down selectSeat' : 'normal-seat-down selectSeat';
          default:
            return selected ? 'normal-seat-selected-down selectSeat' : 'normal-seat-down selectSeat';
          }
        } else if (type === 2) {
          switch (orientation) {
          case 'l':
            return selected ? 'senior-seat-selected-left selectSeat' : 'senior-seat-left selectSeat';
          case 'r':
            return selected ? 'senior-seat-selected-right selectSeat' : 'senior-seat-right selectSeat';
          case 'u':
            return selected ? 'senior-seat-selected-up selectSeat' : 'senior-seat-up selectSeat';
          case 'd':
            return selected ? 'senior-seat-selected-down selectSeat' : 'senior-seat-down selectSeat';
          default:
            return selected ? 'senior-seat-selected-down selectSeat' : 'senior-seat-down selectSeat';
          }
        } else return '';
      };
    },
    /** 校验 */
    // 主席台只能添加一个
    containRostrum() {
      let isContain = false;
      this.seatTemplate.facilities.forEach((item) => {
        if (item.facilityType === 'rostrum') {
          isContain = true;
        }
      });
      return isContain;
    },
    showMultiSeatsPop: {
      get() {
        return this.showEditForm && this.editInfo.type.indexOf('座位') !== -1;
      },
      set() { this.val = false; },
    },

    showFacilityPop: {
      get() {
        return this.showEditForm && this.editInfo.type.indexOf('座位') === -1;
      },
      set() { this.val = false; },
    },
  },

  data() {
    // 校验布局名
    const validateName = (rule, value, callback) => {
      const blankStr = new RegExp(/\s+/g);
      if (value === '') {
        callback(new Error('请输入布局名称'));
      } else if (blankStr.test(value)) {
        callback(new Error('布局名称不能包含空格'));
      } else if (this.seatTemplateNameArr
      && this.seatTemplateNameArr.indexOf(value) !== -1) {
        callback(new Error('布局名称不能重复'));
      } else {
        callback();
      }
    };
    const validateHoldRostrum = (rule, value, callback) => {
      if (!/^(0|[1-9][0-9]*)$/.test(value) || value > 12 || value < 0) {
        callback(new Error('请输入0-12的整数'));
      } else if (this.facilityInfo.facilityType === 'rostrum' && this.facilityInfo.facilityWidth < value) {
        callback(new Error('座位数量须大于等于0，小于等于长度!'));
      } else if (this.facilityInfo.facilityType === 'rostrum' && value > 0
       && (this.facilityInfo.facilityWidth + value) % 2 !== 0) {
        callback(new Error('座位数量须与主席台长度奇偶一致!'));
      } else {
        callback();
      }
    };
    // 校验设施的高度（会议桌）
    const validateFacilityHeight = (rule, value, callback) => {
      if (this.facilityInfo.facilityType === 'table' && (!/^\d+$/.test(value) || value > 12 || value < 1)) {
        callback(new Error('请输入1-12的整数'));
      } else {
        callback();
      }
    };
    // 校验主席台高度
    const validateRostrumHeight = (rule, value, callback) => {
      // 主席台高度(1-8)
      if (this.facilityInfo.facilityType === 'rostrum' && (!/^\d+$/.test(value) || value > 8 || value < 1)) {
        callback(new Error('请输入1-8的整数'));
      } else {
        callback();
      }
    };
    // 校验设施长度
    const validateFacilityWidth = (rule, value, callback) => {
      // 主屏幕长度(3-10)
      if (this.facilityInfo.facilityType === 'mainScreen' && (!/^\d+$/.test(value) || value > 10 || value < 3)) {
        callback(new Error('请输入3-10的整数'));
        // 主席台和会议桌 (1-12)
      } else if (!/^\d+$/.test(value) || value > 12 || value < 1) {
        callback(new Error('请输入1-12的整数'));
      } else {
        callback();
      }
    };
    // 校验模型横向座位数
    const validateHorizontalSeatNum = (rule, value, callback) => {
      if (this.seatModelInfo.seatModelType === 'uShape' && (!/^\d+$/.test(value) || value > 13 || value < 1)) {
        callback(new Error('请输入1-13的整数'));
      } else if (this.seatModelInfo.seatModelType === 'gyrationShape' && (!/^\d+$/.test(value) || value > 14 || value < 1)) {
        callback(new Error('请输入1-14的整数'));
      } else if (this.seatModelInfo.seatModelType === 'deskShape' && (!/^\d+$/.test(value) || value > 16 || value < 1)) {
        callback(new Error('请输入1-16的整数'));
      } else {
        callback();
      }
    };
    // 校验模型纵向座位数
    const validateVerticalSeatNum = (rule, value, callback) => {
      if (this.seatModelInfo.seatModelType === 'uShape' && (!/^\d+$/.test(value) || value > 15 || value < 1)) {
        callback(new Error('请输入1-15的整数'));
      } else if (this.seatModelInfo.seatModelType === 'gyrationShape' && (!/^\d+$/.test(value) || value > 10 || value < 1)) {
        callback(new Error('请输入1-10的整数'));
      } else if (this.seatModelInfo.seatModelType === 'deskShape' && (!/^\d+$/.test(value) || value > 12 || value < 1)) {
        callback(new Error('请输入1-12的整数'));
      } else {
        callback();
      }
    };

    return {
      nameEdit: false,
      // 框选
      reactArea: {
        startX: 0,
        startY: 0,
        endX: 0,
        endY: 0,
        scrollTop: 0,
      },
      isMouseDown: false,
      areaSelect: null,
      // 设施座位不可放置状态
      forbidden: false,
      // 座位编辑div 显示
      editDivStyle: null,
      // 网格的左+右边框宽度
      doubleBorder: 10,
      // 是否可以添加座位
      enableAddSeat: false,
      // 设施鼠标事件可用状态
      enableEvent: true,
      // 布局行数
      rows: 20,
      // 布局列数
      columns: 24,
      // 布局格子的大小
      gridSize: 0,
      // 模型选择对话框显示
      dialogModelSelectVisible: true,
      // 模型编辑对话框显示
      dialogModelInfoVisible: false,
      // 添加设施对话框显示
      dialogFacilityVisible: false,
      // 设施对话框显示
      dialogFacilityDetailVisible: false,
      // 编辑座位设施详细信息
      showEditForm: false,
      // 可移动状态的模型显示
      showDraggModel: false,
      // 可移动状态的设施
      showDraggFacility: false,
      // 添加座位模型时记录的座位模型信息
      seatModelInfo: {
        modelName: '', // 模型名称
        seatModelType: '', // 座位模型('uShape','gyrationShape','deskShape,'customize')
        verticalSeatNum: 4, // 纵向座位数
        horizontalSeatNum: 4, // 横向座位数:
        orientation: 'u', // 朝向（上下左右 一次 u d l r）默认朝上,u型以外的模型默认为空
        circleNum: 1, // 有多少环，最多支持三环
      },
      // 座位模型数组（临时的，当点击确定时将值赋给gridArray）
      seatModelArray: [],
      // 添加设施时记录的当前要添加的设施信息
      facilityInfo: {
        id: 0, // 设施id（用来区分相同设施，便于编辑删除）
        topIndex: 0, // 设施顶部位于布局的第几格
        leftIndex: 0, // 设施左边部分位于布局的第几格
        facilityWidth: 0, // 设施横向占多少格
        facilityHeight: 0, // 设施纵向占多少格
        facilityType: '', // 设施类型 'table'桌子,'rostrum' 主席台,'mainScreen' 主屏幕,'door' 入口
        holdSeatNum: 0, // 设施容纳的座位数量
        rostrumHeight: 1, // 主席台高度
        facilityOrientation: 'u', // 设施朝（上下左右 依次 u d l r）默认朝上
      },

      // 座位布局信息
      seatTemplate: {
        /*
          {
            二维数组中每个元素存储以下信息
            type：'普通座位':1,高管座：2  设施：4 围墙 9，围墙上的门：8
            orientation：座位或者设施的朝向
            id: 设施存设施id，座位存参会人（没有为空）
            label：编号
          }
        */
        updateTime: '', // 按时间排序
        layoutArray: [], // 布局的二维数组
        facilities: [], // 设施信息
        name: '', // 布局名称
        id: '', // 布局id
        wallWidth: 0, // 围墙的长
        wallHeight: 0, // 围墙的高
        wallLeft: 0, // 围墙Left位置
        wallTop: 0, // 围墙Top位置
        doorArea: [], // 门在围墙上的位置 {}
      },
      editInfo: {
        type: '',
        top: 0,
        left: 0,
        Orientation: '',
        width: 0,
        height: 0,
        id: '',
      },
      // 编辑座位类型
      seatType: 1,
      // 座位类型
      seatTypes: [{
        value: 1,
        label: '普通座',
      }, {
        value: 2,
        label: '高管座',
      }],
      // 座位方向 编辑座位时候用
      seatOrientation: 'u',
      // 已有的布局名称
      seatTemplateNameArr: [],
      rules: {
        name: [
          { required: false, validator: validateName, trigger: 'change' },
        ],
      },
      facilityRules: {
        facilityWidth: [
          { required: false, validator: validateFacilityWidth, trigger: 'change' },
          // { type: 'number',  message:'请输入整数',trigger: 'change' },
        ],
        facilityHeight: [
          { required: false, validator: validateFacilityHeight, trigger: 'change' },
          //  { type: 'number',  message:'请输入1-12的整数',trigger: 'change' },
        ],
        rostrumHeight: [
          { required: false, validator: validateRostrumHeight, trigger: 'change' },
          //  { type: 'number',  message:'请输入1-8的整数',trigger: 'change' },

        ],
        holdSeatNum: [
          { required: false, validator: validateHoldRostrum, trigger: 'change' },
          //  { type: 'number',  message:'请输入0-12的整数',trigger: 'change' },
        ],
      },
      seatModelRules: {
        verticalSeatNum: [
          { required: false, validator: validateVerticalSeatNum, trigger: 'change' },
          //  { type: 'number',  message:'请输入整数',trigger: 'change' },
        ],
        horizontalSeatNum: [
          { required: false, validator: validateHorizontalSeatNum, trigger: 'change' },
          //  { type: 'number',  message:'请输入整数',trigger: 'change' },
        ],
      },
    };
  },

  methods: {

    // 判断是否IE（ie11）
    isIE() {
      if (!!window.ActiveXObject || 'ActiveXObject' in window) {
        return true;
      }
      return false;
    },

    // 计算围墙
    computeWall() {
      this.isIE();
      // 围墙的上下左右
      let top = 0;
      let bottom = 0;
      let left = 0;
      let right = 0;

      if (this.seatTemplate.layoutArray) {
        const rows = this.seatTemplate.layoutArray.length;
        const cols = this.seatTemplate.layoutArray[0].length;
        // 循环获取Top位置
        for (let i = 0; i < rows; i += 1) {
          let temp = -1;
          for (let j = 0; j < cols; j += 1) {
            if (this.seatTemplate.layoutArray[i][j].type !== 0) {
              top = i;
              temp = i;
              break;
            }
          }
          if (temp !== -1) break;
        }

        // 循环获取bottom位置
        for (let i = rows - 1; i > 0; i -= 1) {
          let temp = -1;
          for (let j = 0; j < cols; j += 1) {
            if (this.seatTemplate.layoutArray[i][j].type !== 0) {
              bottom = i;
              temp = i;
              break;
            }
          }
          if (temp !== -1) break;
        }

        // 循环获取left位置
        for (let i = 0; i < cols; i += 1) {
          let temp = -1;
          for (let j = 0; j < rows; j += 1) {
            if (this.seatTemplate.layoutArray[j][i].type !== 0) {
              left = i;
              temp = i;
              break;
            }
          }
          if (temp !== -1) break;
        }

        // 循环获取right位置
        for (let i = cols - 1; i > 0; i -= 1) {
          let temp = -1;
          for (let j = 0; j < rows; j += 1) {
            if (this.seatTemplate.layoutArray[j][i].type !== 0) {
              right = i;
              temp = i;
              break;
            }
          }
          if (temp !== -1) break;
        }
        // 围墙变化清空门
        if (this.seatTemplate.wallWidth !== right - left + 5
        || this.seatTemplate.wallHeight !== bottom - top + 5
        || this.seatTemplate.wallTop !== top - 2 || this.seatTemplate.wallLeft !== left - 2) {
          for (let i = 0; i < this.seatTemplate.facilities.length; i += 1) {
            if (this.seatTemplate.facilities[i].facilityType === 'door') {
              this.seatTemplate.facilities.splice(i, 1);
              i -= 1;
            }
          }
          this.seatTemplate.doorArea = [];
        }
        // 布局为空时
        if (right === 0 && left === 0 && bottom === 0 && top === 0) {
          this.seatTemplate.wallWidth = 0;
          this.seatTemplate.wallHeight = 0;
          this.seatTemplate.wallTop = 0;
          this.seatTemplate.wallLeft = 0;
        } else {
          this.seatTemplate.wallWidth = right - left + 5;// 围墙长度
          this.seatTemplate.wallHeight = bottom - top + 5;// 围墙高度
          this.seatTemplate.wallTop = top - 2;//
          this.seatTemplate.wallLeft = left - 2;
        }
      }
    },
    // 关联校验（主席台座位数小于主席台长度）
    checkHoldSeats() {
      if (this.facilityInfo.facilityType === 'rostrum') {
        this.$refs.facilityFormRef.validateField('holdSeatNum');
      }
    },
    // 打开模型选择
    selModel() {
      this.dialogModelSelectVisible = true;
    },

    // 模型选择
    nextModel(type) {
      // 选择模型 的对话框关闭
      this.dialogModelSelectVisible = false;
      this.seatModelInfo.seatModelType = type;
      // 不是自定义类型要弹出设置模型参数对话框
      if (type !== 'customize') {
        this.dialogModelInfoVisible = true;
      }
      // 回型 不设置方向
      if (type === 'gyrationShape') {
        this.seatModelInfo.orientation = '';
      } else {
        this.seatModelInfo.orientation = 'u';
      }
    },
    // 返回模型选择
    upModel() {
      this.dialogModelInfoVisible = false;
      this.dialogModelSelectVisible = true;
    },
    // 确定模型
    okModel(e) {
      // 关闭设施设置对话框
      this.$refs.seatModelFormRef.validate((valid) => {
        if (valid) {
        // 关闭对话框
          this.showDraggModel = true;
          this.dialogModelInfoVisible = false;
          // 添加模型时 禁用 设施的鼠标事件（确定位置后开启）
          this.enableEvent = false;
          // 初始化位置
          this.$nextTick(() => {
            const newWidth = this.seatModelArrayWidth * (this.gridSize + this.doubleBorder);
            const newHeight = this.seatModelArrayHeight * (this.gridSize + this.doubleBorder);
            const left = e.clientX - this.$refs.seatWrapperRef.offsetLeft - Math.ceil(newWidth / 2) - $('.menu-expanded').width() - 32;
            const top = e.clientY - this.$refs.seatWrapperRef.offsetTop - Math.ceil(newHeight / 2)
          - $('.warp-breadcrum').height() - 32 + this.$refs.mainSeatAreaRef.scrollTop;
            this.modifyStyle(this.$refs.seatsRef.$el.style, `${newWidth}px`, `${newHeight}px`, `${top}px`, `${left}px`);
          });
        }
      });
    },
    // 添加设施
    addFacility() {
      this.dialogFacilityVisible = true;
      this.dialogFacilityDetailVisible = false;
    },

    // 设施选择
    nextFacility(type) {
      if (type === 'rostrum' && this.containRostrum) {
        return;
      }
      if (type === 'door' && this.seatTemplate.wallWidth === 0) {
        return;
      }
      this.editInfo.id = '';
      // 选择设施 的对话框关闭
      this.dialogFacilityVisible = false;
      // this.seatModelInfo.seatModelType = type
      this.facilityInfo.facilityType = type;
      // 除了会议桌不需要设置朝向  添加默认值
      if (type === 'door') {
        this.facilityInfo.facilityWidth = 2;
        this.facilityInfo.facilityHeight = 1;
        this.facilityInfo.holdSeatNum = 0;
        this.facilityInfo.facilityOrientation = 'u';
      } else if (type === 'mainScreen') {
        this.facilityInfo.facilityHeight = 1;
        this.facilityInfo.facilityWidth = 3;
        this.facilityInfo.holdSeatNum = 0;
        this.facilityInfo.facilityOrientation = 'u';
      } else if (type === 'rostrum') {
        this.facilityInfo.facilityWidth = 3;
        this.facilityInfo.facilityHeight = 2;
        this.facilityInfo.holdSeatNum = 1;
        this.facilityInfo.rostrumHeight = 1;
        this.facilityInfo.facilityOrientation = 'u';
      } else if (type === 'table') {
        // 会议桌不用设置方向
        this.facilityInfo.holdSeatNum = 0;
        this.facilityInfo.facilityOrientation = '';
        this.facilityInfo.facilityWidth = 4;
        this.facilityInfo.facilityHeight = 4;
      }
      // 弹出 设施参数设置对话框
      this.dialogFacilityDetailVisible = true;
    },

    // 返回设施选择
    upFacility() {
      this.dialogFacilityVisible = true;
      this.dialogFacilityDetailVisible = false;
    },
    // 确定添加设施
    okFacility(e) {
      // 关闭设施设置对话框
      this.$refs.facilityFormRef.validate((valid) => {
        if (valid) {
          // new 一个设施
          if (this.editInfo.id !== '') {
            this.removeFacilities();
            for (let i = 0; i < this.seatTemplate.facilities.length; i += 1) {
              if (this.seatTemplate.facilities[i].id === this.editInfo.id) {
                this.facilityInfo = this.seatTemplate.facilities[i];
                this.editInfo.id = '';
                break;
              }
            }
          }
          this.dialogFacilityDetailVisible = false;
          this.showDraggFacility = true;
          this.enableEvent = false;// 添加设施时禁用已有设施的点击编辑事件
          // 初始化位置
          this.$nextTick(() => {
            const newWidth = this.facilityInfo.facilityWidth * (this.gridSize + this.doubleBorder);
            const newHeight = this.facilityInfo.facilityHeight
                              * (this.gridSize + this.doubleBorder);
            const left = e.clientX - this.$refs.seatWrapperRef.offsetLeft - Math.ceil(newWidth / 2) - $('.menu-expanded').width() - 32;
            const top = e.clientY - this.$refs.seatWrapperRef.offsetTop - Math.ceil(newHeight / 2)
        - $('.warp-breadcrum').height() - 32 + this.$refs.mainSeatAreaRef.scrollTop;
            this.modifyStyle(this.$refs.facilityRef.$el.style, '', '', `${top}px`, `${left}px`);
          });
        }
      });
    },
    // 添加单个座位
    addSeat() {
      this.cancelSelected();// 取消所有正在编辑的selected 状态
      this.$message({
        message: '点击画布外，取消添加座位',
        center: true,
        duration: 10000,
        showClose: true,
        customClass: 'custom-msg',
      });

      this.enableEvent = false;
      this.enableAddSeat = true;
    },
    // 清空全部
    clearAll() {
      this.$confirm('此操作将清空当前画布全部内容，是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        // 点击确定的操作  清空画布
        this.resetSeat();
        this.seatTemplate.facilities = [];
      }).catch(() => {
        // 点取消的提示
      });
      this.enableAddSeat = false;
    },
    // 清空校验
    clearValidates(formName) {
      if (this.$refs[formName]) {
        this.$refs[formName].clearValidate();
      }
    },

    // 重置座位
    resetSeat() {
      // 将所有座位的值变为0
      const oldArray = this.seatTemplate.layoutArray.slice();
      for (let i = 0; i < this.rows; i += 1) {
        for (let j = 0; j < this.columns; j += 1) {
          oldArray[i][j] = {
            type: 0, id: '', label: '', orientation: '',
          };
        }
      }
      this.seatTemplate.layoutArray = oldArray;
      this.seatTemplate.wallHeight = 0;
      this.seatTemplate.wallWidth = 0;
    },
    // 点击编辑设施
    editFacility(fac) {
      // 在指定位置 显示出编辑框
      this.cancelSelected();// 先取消所有的编辑select 状态
      fac.selected = true;

      const top = (this.gridSize + this.doubleBorder) * fac.topIndex
      + this.diffLeftTop(fac.facilityOrientation,
        fac.facilityWidth, fac.facilityHeight, this.gridSize + this.doubleBorder);
      const left = (this.gridSize + this.doubleBorder) * fac.leftIndex
                  - this.diffLeftTop(fac.facilityOrientation,
                    fac.facilityWidth, fac.facilityHeight, this.gridSize + this.doubleBorder);
      const width = fac.facilityWidth * (this.gridSize + this.doubleBorder);
      const height = fac.facilityHeight * (this.gridSize + this.doubleBorder);

      // 刷新pop弹出的编辑框位置
      this.showEditForm = false;
      this.$nextTick(() => {
        this.modifyStyle(this.$refs.popFacilityEditDiv.style, `${width}px`, `${height}px`, `${top}px`, `${left}px`);
        if (fac.facilityOrientation === 'l' || fac.facilityOrientation === 'r') {
          this.$refs.popFacilityEditDiv.style.webkitTransform = 'rotate(90deg)';
          this.$refs.popFacilityEditDiv.style.mozTransform = 'rotate(90deg)';
          this.$refs.popFacilityEditDiv.style.msTransform = 'rotate(90deg)';
          this.$refs.popFacilityEditDiv.style.oTransform = 'rotate(90deg)';
          this.$refs.popFacilityEditDiv.style.transform = 'rotate(90deg)';
        }
        if (fac.facilityType === 'door') {
          this.editInfo.type = '入口';
        } else if (fac.facilityType === 'mainScreen') {
          this.editInfo.type = '主屏幕';
        } else if (fac.facilityType === 'rostrum') {
          this.editInfo.type = '主席台';
        } else if (fac.facilityType === 'table') {
          this.editInfo.type = '会议桌';
        } else {
          this.editInfo.type = '';
        }
        this.facilityInfo = JSON.parse(JSON.stringify(fac));
        this.editInfo.id = fac.id;
        this.showEditForm = true;
      });
    },
    // 删除选中的设施或座位
    deleteItems() {
      // 如果编辑的是座位 直接遍历二维数组
      if (this.editInfo.type === '座位') { // 删除单个座位
        this.seatTemplate.layoutArray[this.editInfo.top][this.editInfo.left] = {
          type: 0, id: '', label: '', orientation: '',
        };
      } else if (this.editInfo.type === '多个座位') { // 删除多个座位
        this.modifySeats('', '', 'delete');
      } else {
        this.removeFacilities();
      }
      this.isMouseDown = false;
      this.showEditForm = false;
    },
    // 编辑座位或者设施 时的确定按钮或编辑设施事件
    okEdit(seatOrientation, seatType) {
      if (this.editInfo.type === '座位') {
        this.seatTemplate.layoutArray[this.editInfo.top][this.editInfo.left] = {
          type: seatType, id: '', label: '', orientation: seatOrientation,
        };
      } else if (this.editInfo.type === '多个座位') {
        this.modifySeats(seatType, seatOrientation);
        this.removeSelectClass();
      } else {
        this.dialogFacilityDetailVisible = true;
      }
      this.isMouseDown = false;
      this.showEditForm = false;
    },

    // 取消编辑选中的状态
    cancelSelected() {
      if (this.editInfo.type === '座位') { // 单个座位
        delete this.seatTemplate.layoutArray[this.editInfo.top][this.editInfo.left].selected;
      } else if (this.editInfo.type === '多个座位') { // 多个座位
        this.removeSelectClass();
      } else { // 设施
        for (let i = 0; i < this.seatTemplate.facilities.length; i += 1) {
          if (this.seatTemplate.facilities[i].id === this.editInfo.id) {
            delete this.seatTemplate.facilities[i].selected;
            this.editInfo.id = '';
            break;
          }
        }
      }
    },
    // 取消编辑
    cancelEdit() {
      this.cancelSelected();
      this.showEditForm = false;
      this.isMouseDown = false;
    },
    // 移除框选状态的样式
    removeSelectClass() {
      const { length } = document.getElementsByClassName('active-seat');
      if (length > 0) {
        for (let i = 0; i < length; i += 1) {
          if (document.getElementsByClassName('active-seat')[i]) {
            this.replaceCss(false, document.getElementsByClassName('active-seat')[i].classList);
            document.getElementsByClassName('active-seat')[i].classList.remove('active-seat');
            i -= 1;
          }
        }
      }
    },
    // 移除设施
    removeFacilities() {
      // 删除设施数组中的该设施对象
      for (let i = 0; i < this.seatTemplate.facilities.length; i += 1) {
        if (this.seatTemplate.facilities[i].id === this.editInfo.id) {
          this.seatTemplate.facilities.splice(i, 1);
          break;
        }
      }
      // 将二维数组修设施占位部分type 值修改为0(此处可后续优化
      for (let i = 0; i < this.rows; i += 1) {
        for (let j = 0; j < this.columns; j += 1) {
          if (this.seatTemplate.layoutArray[i][j].id === this.editInfo.id) {
            this.seatTemplate.layoutArray[i][j] = {
              type: 0, id: '', label: '', orientation: '',
            };
          }
        }
      }
      // 如果是入口
      if (this.editInfo.type === '入口') {
        for (let i = 0; i < this.seatTemplate.doorArea.length; i += 1) {
          if (this.seatTemplate.doorArea[i].id === this.editInfo.id) {
            this.seatTemplate.doorArea.splice(i, 1);
            i -= 1;
          }
        }
      }
      this.editInfo.id = '';
    },
    // 编辑多个座位
    modifySeats(seatType, seatOrientation, operation = 'update') {
      const { length } = document.getElementsByClassName('active-seat');
      if (length > 0) {
        for (let i = 0; i < length; i += 1) {
          const row = document.getElementsByClassName('active-seat')[i].attributes.row.nodeValue;
          const column = document.getElementsByClassName('active-seat')[i].attributes.column.nodeValue;
          if (operation === 'delete') {
            this.seatTemplate.layoutArray[row][column] = {
              type: 0, id: '', label: '', orientation: '',
            };
          } else {
            if (seatType !== '') {
              this.seatTemplate.layoutArray[row][column].type = seatType;
            }
            if (seatOrientation !== '') {
              this.seatTemplate.layoutArray[row][column].orientation = seatOrientation;
            }
          }
        }
      }
    },

    // 结束编辑状态
    endEditState() {
      this.cancelSelected();// 取消所有正在编辑的selected 状态
      this.enableAddSeat = false;
      this.showDraggModel = false;
      this.showDraggFacility = false;
      this.showEditForm = false;
      this.isMouseDown = false;
      this.enableEvent = true;
    },
    // 取消模型
    cancel() {
      this.$confirm('是否取消保存布局图，是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        localStorage.removeItem('seatTemplate');
        this.$router.go(-1);
      }).catch(() => {
        // 点取消的提示
      });
    },
    // 保存并退出
    saveAndQuit() {
      // 如果布局为空不能保存
      if (this.seatTemplate.wallWidth === 0) {
        this.$message({
          message: '没有内容，请修改后保存',
          center: true,
          duration: 3000,
          customClass: 'custom-msg',
        });
        return;
      }
      this.$refs.templateform.validate((valid) => {
        if (valid) {
          let no = 1;
          // 给座位附上编号
          for (let i = 0; i < this.rows; i += 1) {
            for (let j = 0; j < this.columns; j += 1) {
              if (this.seatTemplate.layoutArray[i][j].type === 1
              || this.seatTemplate.layoutArray[i][j].type === 2) {
                this.seatTemplate.layoutArray[i][j].id = no.toString();
                no += 1;
              }
            }
          }
          this.seatTemplate.updateTime = new Date().getTime();
          localStorage.setItem('seatTemplate', JSON.stringify(this.seatTemplate));
          this.$router.go(-1);
        }
      });
    },
    // 添加门
    addDoor(rowIndex, colIndex, position) {
      // 不是处于添加设施状态返回
      if (!this.showDraggFacility) {
        return;
      }
      // 门的位置
      const topIndex = this.seatTemplate.wallTop + rowIndex;
      const leftIndex = this.seatTemplate.wallLeft + colIndex;
      // 朝上或者朝下方向的门 只能添加在围墙的上方或者下方
      if ((position === 'horizontalsectionleft' || position === 'horizontalsectionright')
          && (this.facilityInfo.facilityOrientation === 'r'
          || this.facilityInfo.facilityOrientation === 'l')) {
        // step 1 校验位置是否可放
        // 如果门的位置在围墙边角 不可放置
        if (rowIndex < 0 || rowIndex + 2 >= this.seatTemplate.wallHeight) {
          this.forbidden = true;
        } else {
          // 已经有门的位置不可放置
          this.seatTemplate.doorArea.forEach((item) => {
            // 门的位置已经有门，
            if ((rowIndex === item.top && leftIndex === item.left)
              || (rowIndex + 1 === item.top && leftIndex === item.left)) {
              this.forbidden = true;
              // console.log('越界');
            }
          });
        }
        if (this.forbidden) {
          return;
        }
        const facilityId = Math.random().toString(36).substr(3, 6);// 生成6位长度的设施ID
        this.seatTemplate.doorArea.push({ id: facilityId, top: rowIndex, left: colIndex });
        this.seatTemplate.doorArea.push({ id: facilityId, top: rowIndex + 1, left: colIndex });
        // step3 将设备信息添加到 设备数组中
        const facilityTmp = {
          id: facilityId,
          topIndex,
          leftIndex,
          rostrumHeight: this.facilityInfo.rostrumHeight,
          facilityWidth: this.facilityInfo.facilityWidth,
          facilityHeight: this.facilityInfo.facilityHeight,
          facilityType: this.facilityInfo.facilityType,
          holdSeatNum: this.facilityInfo.holdSeatNum,
          facilityOrientation: this.facilityInfo.facilityOrientation,
        };
        this.seatTemplate.facilities.push(facilityTmp);
        this.showDraggFacility = false;
        this.enableEvent = true;// 开启设施点击事件
      }
      if ((position === 'verticalsectionup' || position === 'verticalsectiondown')
          && (this.facilityInfo.facilityOrientation === 'u' || this.facilityInfo.facilityOrientation === 'd')) {
        // step 1 校验位置是否可放
        // 门的位置已经有门，或者门的位置添加超出了围墙边角则不能放置

        if (colIndex < 0 || colIndex + 2 >= this.seatTemplate.wallWidth) {
          this.forbidden = true;
        } else {
          this.seatTemplate.doorArea.forEach((item) => {
            if ((colIndex === item.left && rowIndex === item.top)
             || (colIndex + 1 === item.left && rowIndex === item.top)) {
              this.forbidden = true;
              // console.log('越界');
            }
          });
        }

        if (this.forbidden) {
          return;
        }
        const facilityId = Math.random().toString(36).substr(3, 6);// 生成6位长度的设施ID
        this.seatTemplate.doorArea.push({ id: facilityId, top: rowIndex, left: colIndex });
        this.seatTemplate.doorArea.push({ id: facilityId, top: rowIndex, left: colIndex + 1 });
        // step3 将设备信息添加到 设备数组中
        const facilityTmp = {
          id: facilityId,
          topIndex,
          leftIndex,
          rostrumHeight: this.facilityInfo.rostrumHeight,
          facilityWidth: this.facilityInfo.facilityWidth,
          facilityHeight: this.facilityInfo.facilityHeight,
          facilityType: this.facilityInfo.facilityType,
          holdSeatNum: this.facilityInfo.holdSeatNum,
          facilityOrientation: this.facilityInfo.facilityOrientation,
        };
        this.seatTemplate.facilities.push(facilityTmp);
        this.showDraggFacility = false;
        this.enableEvent = true;// 开启设施点击事件
      }
    },
    // 初始化添加座位模型时模型数组(子组件seats传过来的座位模型值)
    initSeatModelArray(arr) {
      this.seatModelArray = arr;
    },

    // 鼠标事件
    // 鼠标移动事件
    mousemove(e) {
      // 座位模型
      if (this.$refs.seatsRef) {
        const newWidth = this.seatModelArrayWidth * (this.gridSize + this.doubleBorder);
        const newHeight = this.seatModelArrayHeight * (this.gridSize + this.doubleBorder);
        const left = e.clientX - this.$refs.seatWrapperRef.offsetLeft - Math.ceil(newWidth / 2) - $('.menu-expanded').width() - 32;
        const top = e.clientY - this.$refs.seatWrapperRef.offsetTop - Math.ceil(newHeight / 2)
        - $('.warp-breadcrum').height() - 32 + this.$refs.mainSeatAreaRef.scrollTop;
        // const style = `left: ${left}px;top: ${top}px;width:${newWidth}px;height:${newHeight}px`;
        // this.$refs.seatsRef.$el.style = style;
        this.modifyStyle(this.$refs.seatsRef.$el.style, `${newWidth}px`, `${newHeight}px`, `${top}px`, `${left}px`);
      } else if (this.$refs.facilityRef) { // 移动状态的设施
        const newWidth = this.facilityInfo.facilityWidth * (this.gridSize + this.doubleBorder);
        const newHeight = this.facilityInfo.facilityHeight * (this.gridSize + this.doubleBorder);
        const left = e.clientX - this.$refs.seatWrapperRef.offsetLeft - Math.ceil(newWidth / 2) - $('.menu-expanded').width() - 32;
        const top = e.clientY - this.$refs.seatWrapperRef.offsetTop - Math.ceil(newHeight / 2) - $('.warp-breadcrum').height()
         - 32 + this.$refs.mainSeatAreaRef.scrollTop;
        // const style = `left: ${left}px;top: ${top}px;width:${newWidth}px;height:${newHeight}px`;
        // this.$refs.facilityRef.$el.style = style;
        this.modifyStyle(this.$refs.facilityRef.$el.style, `${newWidth}px`, `${newHeight}px`, `${top}px`, `${left}px`);
      } else if (this.enableAddSeat) { // 添加单个座位
        const row = e.target.getAttribute('row');
        if (!row) {
          return;
        }
        const top = e.target.offsetTop;
        const left = e.target.offsetLeft;
        // const style = `left: ${left - 6}px;top: ${top - 6}px;`;
        // this.$refs.addSeatDivRef.style = style;
        this.modifyStyle(this.$refs.addSeatDivRef.style, '', '', `${top - 11}px`, `${left - 11}px`);
      } else if (this.isMouseDown && !this.editState) {
        this.reactArea.endX = e.clientX;
        this.reactArea.endY = e.clientY - this.reactArea.scrollTop
        + this.$refs.mainSeatAreaRef.scrollTop;
        let leftValue = 0;
        let topValue = 0;
        const widthValue = Math.abs(this.reactArea.startX - this.reactArea.endX);
        const heightValue = Math.abs(this.reactArea.startY - this.reactArea.endY);
        if (this.reactArea.startX >= this.reactArea.endX) {
          leftValue = this.reactArea.endX - this.$refs.seatWrapperRef.offsetLeft - $('.menu-expanded').width() - 32;
        } else {
          leftValue = this.reactArea.startX - this.$refs.seatWrapperRef.offsetLeft - $('.menu-expanded').width() - 32;
        }
        if (this.reactArea.startY > this.reactArea.endY) {
          topValue = this.reactArea.endY - this.$refs.seatWrapperRef.offsetTop + this.reactArea.scrollTop - $('.warp-breadcrum').height() - 50;
        } else {
          topValue = this.reactArea.startY - this.$refs.seatWrapperRef.offsetTop + this.reactArea.scrollTop - $('.warp-breadcrum').height() - 50;
        }
        // 判断同时有宽高才开始画虚线框

        if (this.reactArea.startX !== this.reactArea.endX
        && this.reactArea.startY !== this.reactArea.endY) {
          // const style = `left: ${leftValue}px;top: ${topValue}px;
          //           width: ${widthValue}px;height: ${heightValue}px;`;
          // this.$refs.selectedArea.style = style;
          this.modifyStyle(this.$refs.selectedArea.style, `${widthValue}px`, `${heightValue}px`, `${topValue}px`, `${leftValue}px`);
          this.modifyStyle(this.$refs.popDiv.style, `${widthValue}px`, `${heightValue}px`, `${topValue}px`, `${leftValue}px`);
        }
        const children = this.$refs.innerSeatWrapperRef.getElementsByClassName('selectSeat');
        const childrenHeight = this.gridSize + 5;
        const childrenWidth = this.gridSize + 5;
        for (let i = 0; i < children.length; i += 1) {
          // 每个元素的位置
          const { offsetLeft } = children[i];
          const { offsetTop } = children[i];
          // 每个li元素的宽高
          const endPositionH = childrenHeight + offsetTop;
          const endPositionW = childrenWidth + offsetLeft;
          // 五个条件满足一个就可以判断被选择
          // 一是右下角在选择区域内
          const require1 = endPositionH > topValue && endPositionW > leftValue
          && endPositionH < topValue + heightValue && endPositionW < leftValue + widthValue;
          // 二是左上角在选择区域内
          const require2 = offsetTop > topValue && offsetLeft > leftValue
          && offsetTop < topValue + heightValue && offsetLeft < leftValue + widthValue;
          // 三是右上角在选择区域内
          const require3 = offsetTop > topValue && offsetLeft + childrenWidth > leftValue
          && offsetTop < topValue + heightValue
          && offsetLeft + childrenWidth < leftValue + widthValue;
          // 四是左下角在选择区域内
          const require4 = offsetTop + childrenHeight > topValue && offsetLeft > leftValue
          && offsetTop + childrenHeight < topValue + heightValue
          && offsetLeft < leftValue + widthValue;
          // 五选择区域在元素体内
          const require5 = offsetTop < topValue && offsetLeft < leftValue
          && offsetTop + childrenHeight > topValue + heightValue
          && offsetLeft + childrenWidth > leftValue + widthValue;
          if (require1 || require2 || require3 || require4 || require5) {
            children[i].classList.add('active-seat');
            this.replaceCss(true, children[i].classList);
          } else {
            children[i].classList.remove('active-seat');
            this.replaceCss(false, children[i].classList);
          }
        }
      }
    },
    // 修改style 属性 target,width,height,top,left 为了兼容ie
    modifyStyle(target, width, height, top, left) {
      if (width !== '') {
        target.width = width;
      }
      if (height !== '') {
        target.height = height;
      }
      if (top !== '') {
        target.top = top;
      }
      if (left !== '') {
        target.left = left;
      }
    },
    // 替换样式（toSelected：true 换成选中状态，false 换成未选中状态
    // target:需要替换样式的目标对象）
    replaceCss(toSelected, target) {
      if (toSelected) {
        if (target.contains('normal-seat-left')) {
          target.remove('normal-seat-left');
          target.add('normal-seat-selected-left');
        }
        if (target.contains('normal-seat-right')) {
          target.remove('normal-seat-right');
          target.add('normal-seat-selected-right');
        }
        if (target.contains('normal-seat-up')) {
          target.remove('normal-seat-up');
          target.add('normal-seat-selected-up');
        }
        if (target.contains('normal-seat-down')) {
          target.remove('normal-seat-down');
          target.add('normal-seat-selected-down');
        }
        if (target.contains('senior-seat-left')) {
          target.remove('senior-seat-left');
          target.add('senior-seat-selected-left');
        }
        if (target.contains('senior-seat-right')) {
          target.remove('senior-seat-right');
          target.add('senior-seat-selected-right');
        }
        if (target.contains('senior-seat-up')) {
          target.remove('senior-seat-up');
          target.add('senior-seat-selected-up');
        }
        if (target.contains('senior-seat-down')) {
          target.remove('senior-seat-down');
          target.add('senior-seat-selected-down');
        }
        // IE11 不支持
        // target.replace('normal-seat-left', 'normal-seat-selected-left');
        // target.replace('normal-seat-right', 'normal-seat-selected-right');
        // target.replace('normal-seat-up', 'normal-seat-selected-up');
        // target.replace('normal-seat-down', 'normal-seat-selected-down');
        // target.replace('senior-seat-left', 'senior-seat-selected-left');
        // target.replace('senior-seat-right', 'senior-seat-selected-right');
        // target.replace('senior-seat-up', 'senior-seat-selected-up');
        // target.replace('senior-seat-down', 'senior-seat-selected-down');
      } else {
        if (target.contains('normal-seat-selected-left')) {
          target.add('normal-seat-left');
          target.remove('normal-seat-selected-left');
        }
        if (target.contains('normal-seat-selected-right')) {
          target.add('normal-seat-right');
          target.remove('normal-seat-selected-right');
        }
        if (target.contains('normal-seat-selected-up')) {
          target.add('normal-seat-up');
          target.remove('normal-seat-selected-up');
        }
        if (target.contains('normal-seat-selected-down')) {
          target.add('normal-seat-down');
          target.remove('normal-seat-selected-down');
        }
        if (target.contains('senior-seat-selected-left')) {
          target.add('senior-seat-left');
          target.remove('senior-seat-selected-left');
        }
        if (target.contains('senior-seat-selected-right')) {
          target.add('senior-seat-right');
          target.remove('senior-seat-selected-right');
        }
        if (target.contains('senior-seat-selected-up')) {
          target.add('senior-seat-up');
          target.remove('senior-seat-selected-up');
        }
        if (target.contains('senior-seat-selected-down')) {
          target.add('senior-seat-down');
          target.remove('senior-seat-selected-down');
        }
        // IE 11 不支持
        // target.replace('normal-seat-selected-left', 'normal-seat-left');
        // target.replace('normal-seat-selected-right', 'normal-seat-right');
        // target.replace('normal-seat-selected-up', 'normal-seat-up');
        // target.replace('normal-seat-selected-down', 'normal-seat-down');
        // target.replace('senior-seat-selected-left', 'senior-seat-left');
        // target.replace('senior-seat-selected-right', 'senior-seat-right');
        // target.replace('senior-seat-selected-up', 'senior-seat-up');
        // target.replace('senior-seat-selected-down', 'senior-seat-down');
      }
    },
    // 鼠标点击事件
    mousedown(e) {
      // this.modifyStyle();
      if (!this.editState && e.target.classList.contains('block-area')) {
        this.reactArea.scrollTop = this.$refs.mainSeatAreaRef.scrollTop;
        this.reactArea.startX = e.clientX;
        this.reactArea.startY = e.clientY;
        this.isMouseDown = true;
      }
      // 判断点击的位置是否在格子上
      const row = e.target.getAttribute('row');
      const column = e.target.getAttribute('column');
      if (!row || !column) {
        this.forbidden = true;// 不可放置
        return;
      }

      // 如果有可以拖动的设施或者模型点击后 确定位置取消拖拽状态
      if (this.showDraggModel) { // 添加模型
        // 计算开始位置
        const topIndex = parseInt(row, 10) - Math.ceil(this.seatModelArrayHeight / 2) + 1;
        const leftIndex = parseInt(column, 10) - Math.ceil(this.seatModelArrayWidth / 2) + 1;
        // 将座位的数据添加到二维数组中

        // step 1 判断围墙是否越界
        if (topIndex < 2 || leftIndex < 2 || topIndex + this.seatModelArrayHeight > this.rows - 2
        || leftIndex + this.seatModelArrayWidth > this.columns - 2
        ) {
          this.forbidden = true;// 不可放置
          // console.log('越界');
          return;
        }

        // step 2 判断放置的位置是否被占(除了围墙)
        for (let i = 0; i < this.seatModelArrayHeight; i += 1) {
          for (let j = 0; j < this.seatModelArrayWidth; j += 1) {
            if (this.seatTemplate.layoutArray[i + topIndex][j + leftIndex].type !== 0
            && this.seatModelArray[i][j].type !== 0) {
              this.forbidden = true;// 不可放置
              return;
            }
          }
        }

        // setp 3 放置座位
        for (let i = 0; i < this.seatModelArrayHeight; i += 1) {
          for (let j = 0; j < this.seatModelArrayWidth; j += 1) {
            // 只修改没有座位的位置
            if (this.seatModelArray[i][j].type !== 0) {
              this.seatTemplate.layoutArray[i + topIndex][j + leftIndex] = {
                type: this.seatModelArray[i][j].type,
                id: '',
                label: '',
                orientation: this.seatModelArray[i][j].orientation,
              };
            }
          }
        }
        // step 4 座位模型添加成功移除拖拽的 模型图片
        this.showDraggModel = false;
        this.enableEvent = true;// 开启设施图片的鼠标事件
      } else if (this.showDraggFacility) { // 如果添加设施
        // 门不在这里添加
        if (this.facilityInfo.facilityType === 'door') {
          this.forbidden = true;// 不可放置
          return;
        }
        // 计算开始位置
        let topIndex = parseInt(row, 10) - Math.ceil(this.facilityInfo.facilityHeight / 2) + 1;
        let leftIndex = parseInt(column, 10) - Math.ceil(this.facilityInfo.facilityWidth / 2) + 1;

        // step 2 判断围墙是否越界
        // 如果是旋转90或这270
        if (this.facilityInfo.facilityOrientation === 'l'
        || this.facilityInfo.facilityOrientation === 'r') {
          // 长度偶数
          if (this.facilityInfo.facilityWidth % 2 === 0) {
            if (this.facilityInfo.facilityHeight < this.facilityInfo.facilityWidth) {
              topIndex = parseInt(row, 10) - Math.ceil(this.facilityInfo.facilityHeight / 2) + 1;
              leftIndex = parseInt(column, 10) - Math.ceil(this.facilityInfo.facilityWidth / 2) + 1;
            } else if (this.facilityInfo.facilityHeight % 2 === 0) {
              topIndex = parseInt(row, 10) - Math.ceil(this.facilityInfo.facilityHeight / 2) + 1;
              leftIndex = parseInt(column, 10) - Math.ceil(this.facilityInfo.facilityWidth / 2) + 1;
            } else {
              topIndex = parseInt(row, 10) - Math.ceil(this.facilityInfo.facilityHeight / 2) + 2;
              leftIndex = parseInt(column, 10) - Math.ceil(this.facilityInfo.facilityWidth / 2);
            }
          } else if (this.facilityInfo.facilityHeight < this.facilityInfo.facilityWidth) {
            if (this.facilityInfo.facilityHeight % 2 === 0) {
              topIndex = parseInt(row, 10) - Math.ceil(this.facilityInfo.facilityHeight / 2);
              leftIndex = parseInt(column, 10) - Math.ceil(this.facilityInfo.facilityWidth / 2) + 2;
            } else {
              topIndex = parseInt(row, 10) - Math.ceil(this.facilityInfo.facilityHeight / 2) + 1;
              leftIndex = parseInt(column, 10) - Math.ceil(this.facilityInfo.facilityWidth / 2) + 1;
            }
          } else if (this.facilityInfo.facilityHeight % 2 === 0) {
            topIndex = parseInt(row, 10) - Math.ceil(this.facilityInfo.facilityHeight / 2) + 1;
            leftIndex = parseInt(column, 10) - Math.ceil(this.facilityInfo.facilityWidth / 2) + 1;
          }

          topIndex -= parseInt((this.facilityInfo.facilityWidth
          - this.facilityInfo.facilityHeight) / 2, 10);

          leftIndex += parseInt((this.facilityInfo.facilityWidth
          - this.facilityInfo.facilityHeight) / 2, 10);

          if (topIndex < 2 || leftIndex < 2 || topIndex
          + this.facilityInfo.facilityWidth > this.rows - 2
                  || leftIndex + this.facilityInfo.facilityHeight > this.columns - 2) {
            this.forbidden = true;
            // console.log('越界');
            return;
          }
          // 校验放下去的位置是否空白
          for (let i = 0; i < this.facilityInfo.facilityHeight; i += 1) {
            for (let j = 0; j < this.facilityInfo.facilityWidth; j += 1) {
              if (this.seatTemplate.layoutArray[j + topIndex][i + leftIndex].type !== 0) {
                // this.showDraggModel = false;
                this.forbidden = true;// 不可放置
                // console.log('越界');
                return;
              }
            }
          }
        } else {
          if (topIndex < 2 || leftIndex < 2
           || topIndex + this.facilityInfo.facilityHeight > this.rows - 2
                || leftIndex + this.facilityInfo.facilityWidth > this.columns - 2) {
            this.forbidden = true;
            // console.log('越界');
            return;
          }
          // 校验放下去的位置是否有空白
          for (let i = 0; i < this.facilityInfo.facilityHeight; i += 1) {
            for (let j = 0; j < this.facilityInfo.facilityWidth; j += 1) {
              if (this.seatTemplate.layoutArray[i + topIndex][j + leftIndex].type !== 0) {
                // this.showDraggModel = false;
                this.forbidden = true;
                // console.log('越界');
                return;
              }
            }
          }
        }

        // step3 将设备信息添加到 设备数组中
        const facilityId = Math.random().toString(36).substr(3, 6);// 生成6位长度的设施ID
        const facilityTmp = {
          id: facilityId,
          topIndex,
          leftIndex,
          rostrumHeight: this.facilityInfo.rostrumHeight,
          facilityWidth: this.facilityInfo.facilityWidth,
          facilityHeight: this.facilityInfo.facilityHeight,
          facilityType: this.facilityInfo.facilityType,
          holdSeatNum: this.facilityInfo.holdSeatNum,
          facilityOrientation: this.facilityInfo.facilityOrientation,
        };
        this.seatTemplate.facilities.push(facilityTmp);

        // step4 将设备的占位信息添加到 布局二维数组中
        // 旋转90度或者270度 width和height 对调
        if (this.facilityInfo.facilityOrientation === 'l' || this.facilityInfo.facilityOrientation === 'r') {
          for (let i = 0; i < this.facilityInfo.facilityHeight; i += 1) {
            for (let j = 0; j < this.facilityInfo.facilityWidth; j += 1) {
              this.seatTemplate.layoutArray[j + topIndex][i + leftIndex] = {
                type: 4,
                id: facilityId,
                label: '',
                orientation: this.facilityInfo.facilityOrientation,
              }; // 设施占位标志（如果有必要 可以根据不同的设施设置不同的值
            }
          }
        } else {
          for (let i = 0; i < this.facilityInfo.facilityHeight; i += 1) {
            for (let j = 0; j < this.facilityInfo.facilityWidth; j += 1) {
              this.seatTemplate.layoutArray[i + topIndex][j + leftIndex] = {
                type: 4,
                id: facilityId,
                label: '',
                orientation: this.facilityInfo.facilityOrientation,
              }; // 设施占位标志（如果有必要 可以根据不同的设施设置不同的值
            }
          }
        }

        // // step 4 计算围墙(添加座位时 围墙与座位之间空一圈)
        // 设备添加成功 移除拖拽的设备图片
        this.showDraggFacility = false;
        this.enableEvent = true;
        // 如果是添加座位
      } else if (this.enableAddSeat) { // 添加单个座位
        // 判断点击的位置是否被占
        if (this.seatTemplate.layoutArray[row][column].type !== 0) {
          this.forbidden = true;
          return;
        }
        // 判断是否越界
        if (row < 2 || column < 2
         || parseInt(row, 10) + 1 > this.rows - 2 || parseInt(column, 10) + 1 > this.columns - 2
        ) {
          this.forbidden = true;
          // console.log('越界');
          return;
        }
        this.$set(this.seatTemplate.layoutArray[row], column, {
          type: 1, id: '', label: '', orientation: 'u',
        });
        // step 3 计算围墙
      } else { // 编辑单个座位
        this.cancelSelected();
        const tempSeat = this.seatTemplate.layoutArray[row][column];
        this.seatTemplate.layoutArray[row][column].selected = true;// 换背景图片
        // 1 普通座位,2 高管座
        if (tempSeat.type === 1 || tempSeat.type === 2) {
          // 切换座位时pop 重新渲染
          this.showEditForm = false;
          this.$nextTick(() => {
            this.modifyStyle(this.$refs.popDiv.style, `${this.gridSize}px`, `${this.gridSize}px`, `${e.target.offsetTop}px`, `${e.target.offsetLeft}px`);
            this.editInfo.type = '座位';
            this.editInfo.top = row;
            this.editInfo.left = column;
            this.editInfo.id = 'NO';
            this.seatType = tempSeat.type;
            this.seatOrientation = tempSeat.orientation;
            this.showEditForm = true;
          });
        } else {
          this.showEditForm = false;
        }
      }
    },
    // 鼠标放开事件（框选时）
    mouseup() {
      this.forbidden = false;
      if (!this.editState) {
        this.isMouseDown = false;
        this.modifyStyle(this.$refs.selectedArea.style, `${0}px`, `${0}px`, `${0}px`, `${0}px`);
        const { length } = document.getElementsByClassName('active-seat');
        if (length > 0) {
          this.editInfo.type = '多个座位';
          this.editInfo.id = 'NOs';
          this.seatType = '';// 默认空没选座位类型
          this.seatOrientation = '';// 默认空没选座位朝向
          this.showEditForm = true;
        }
      }
    },
    editName() {
      this.nameEdit = true;
      this.$nextTick(() => {
        this.$refs.nameInput.focus();
      });
    },
    // 初始座位数组
    initSeatArray() {
      // 初始化布局
      const gridArray = Array(this.rows).fill({
        type: 0, id: '', label: '', orientation: '',
      })
        .map(() => Array(this.columns).fill({
          type: 0, id: '', label: '', orientation: '',
        }));
      this.seatTemplate.layoutArray = gridArray;
      // 计算每个格子的尺寸  （画布的长度-（横向格子数+1）*（margin-left+margin-right））/横向格子数
      // this.gridSize = parseInt(
      //   window.getComputedStyle(this.$refs.innerSeatWrapperRef).width, 10,
      // ) / this.columns - this.doubleBorder;
      this.gridSize = 32;
    },

    // 设置是否监听浏览器刷新时弹窗提示
    setIsBeforeunload(val) {
      this.$store.commit('breadcrumb/setIsBeforeunload', val);
    },
  },

  created() {
    // 根据参数判断是编辑还是新增
    const template = this.$route.params.seatTemplateItem == null
      ? [] : Object.keys(this.$route.params.seatTemplateItem);
    this.seatTemplateNameArr = this.$route.params.layoutNameArr;
    if (template.length === 0) { // 新增
      const tempName = this.$route.params.seatTemplateName;
      this.seatTemplate.name = tempName == null ? '' : tempName;
      // 初始化 布局二维数组
      this.initSeatArray();
    } else { // 编辑
      this.gridSize = 32;
      this.dialogModelSelectVisible = false;
      this.seatTemplate = JSON.parse(JSON.stringify(this.$route.params.seatTemplateItem));
      // 编辑的时候校验名称是否重复 除去自身
      const index = this.seatTemplateNameArr.indexOf(this.seatTemplate.name);
      if (index !== -1) {
        this.seatTemplateNameArr.splice(index, 1);
      }
    }
  },

  mounted() {
    // 初始化遮罩层大小
    this.maskStyle = `{ height: ${document.body.clientHeight}px;
    height: ${document.body.clientHeight}px;}`;
    //
    this.clearValidates('templateform');
  },
  beforeRouteEnter(to, from, next) {
    next((vm) => {
      vm.setIsBeforeunload(true);
    });
  },
  beforeRouteLeave(to, from, next) {
    this.setIsBeforeunload(false);
    next();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
@green: #0DAF9C;
.active-seat {
  // background-color: rgba(13, 175, 156, 0.15) !important;
}
.name-edit-icon {
  color: @green;
  margin-left: 12px;
  cursor: pointer;
}
/deep/ .el-card__body{
  padding: 15px 15px 0px 0px;
}
/deep/.v-selected-area {
    position: absolute;
    border: 1px dashed grey;
    z-index: 2100;
}
.vue-input /deep/ .el-input__inner {
    border: 1px solid #fff !important;
    padding-left: 0;
    }
    .vue-input /deep/.el-input{
      width: 280px;
    }

    .param-input/deep/.el-input__inner {
      width: 68px;
    }
    /deep/.el-dialog__body {
    padding: 20px 15px 30px 15px;
    }
    /deep/.el-form-item__label{
      line-height: 40px;
      padding-left:7px;
    }
    /deep/.el-dialog__footer {
    padding: 15px 20px 35px;
}

   /* 编辑操作时的 背景框 */
  .mask {
    width: 100%;
    height: 100%;
     opacity: 0.0;
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
    overflow: hidden;
  }
  .mainSeatArea{
    flex: 1;
    display: flex;
    flex-direction: column;
    width: 100%;
    overflow: auto;
  }
  .title-wrapper{
    width:1060px;
    margin: 0px auto;
    height:40px;
  }

.btn-wrapper{
    display: flex;
    width:1060px;
    margin-left: 80px;
    height:80px;
    margin-bottom: 15px;
    text-align: left;
    justify-content: space-between;
    align-items: center;
    flex-shrink:0;
    button{
      width: 117px;
      height: 36px;
      padding: 10px 20px;
    }
    .addSeat {
      color: @green;
      background: #fff;
    }
    .clear-all {
      color: rgb(51, 62, 94);
      width: 80px;
      padding: 0;
      font-size: 12px;
    }
  }
  .div-icon{
    display: flex;
    font-size: 12px;
    margin-top: 15px;
    div{width: 12px; height: 12px;margin:0px 10px 0px;}
    :nth-child(1) {background: #AFCBE3; }
    :nth-child(3){background: #E7D6C5; }
    :nth-child(5){background: #A7A9B5; }
    span{line-height: 10px;}
  }
  .seat-wrapper{
    // height: 1020px;
    width:1060px;
    height:900px;
    border: 1px solid @green;
    margin-left: 80px;
    position:relative;
    background: #FCFCFC;
    border-radius: 2px;
     flex-grow:1;
       z-index: 1001;
    margin-bottom: 40px;
    flex-shrink: 0;
  }
  /*画布区域大小*/
  .inner-seat-wrapper{
    width:1008px;
    margin: 15px 25px ;
    top:10px;
    position: relative;
    bottom:0px;
    box-sizing: content-box;
    z-index: 1000;
    background: #FCFCFC;
  }
  /*画布中小正方形 */
  .square-block{
    margin:5px;
    background: #F2F4F5;
  }
  .el-col_empty{
    min-height:1px;
  }
 .normal-seat-left{
     background: url('../../../assets/image/web/seat/normal-seat-left.svg')
    center center no-repeat;
    background-size: 32px  32px;
  }
 .normal-seat-right{
    background: url('../../../assets/image/web/seat/normal-seat-right.svg')
      center center no-repeat;
    background-size: 32px  32px;
  }
   .normal-seat-up{
    background: url('../../../assets/image/web/seat/normal-seat-up.svg')
     center center no-repeat;
    background-size: 32px  32px;
  }
 .normal-seat-down{
    background: url('../../../assets/image/web/seat/normal-seat-down.svg')
    center center no-repeat;
    background-size: 32px  32px;
  }
   .normal-seat-selected-left{
     background: url('../../../assets/image/web/seat/normal-seat-selected-left.svg')
     center center no-repeat;
    background-size: 32px 32px;
  }
 .normal-seat-selected-right{
    background: url('../../../assets/image/web/seat/normal-seat-selected-right.svg')
    center center no-repeat;
    background-size: 32px  32px;
  }
   .normal-seat-selected-up{
    background: url('../../../assets/image/web/seat/normal-seat-selected-up.svg')
    center center no-repeat;
    background-size: 32px  32px;
  }
 .normal-seat-selected-down{
    background: url('../../../assets/image/web/seat/normal-seat-selected-down.svg')
    center center no-repeat;
    background-size: 32px  32px;
  }
  .senior-seat-left{
    background: url('../../../assets/image/web/seat/senior-seat-left.svg')
    center center no-repeat;
    background-size: 32px  32px;
  }
 .senior-seat-right{
    background: url('../../../assets/image/web/seat/senior-seat-right.svg')
    center center no-repeat;
    background-size: 32px  32px;

  }
  .senior-seat-down{
    background: url('../../../assets/image/web/seat/senior-seat-down.svg')
    center center no-repeat;
    background-size: 32px  32px;

  }
  .senior-seat-up{
    background: url('../../../assets/image/web/seat/senior-seat-up.svg')
    center center no-repeat;
    background-size: 32px  32px;

  }
 .senior-seat-selected-left{
    background: url('../../../assets/image/web/seat/senior-seat-selected-left.svg')
    center center no-repeat;
    background-size: 32px  32px;

  }
 .senior-seat-selected-right{
    background: url('../../../assets/image/web/seat/senior-seat-selected-right.svg')
    center center no-repeat;
    background-size: 32px  32px;

  }
  .senior-seat-selected-down{
    background: url('../../../assets/image/web/seat/senior-seat-selected-down.svg')
    center center no-repeat;
    background-size: 32px  32px;

  }
 .senior-seat-selected-up{
    background: url('../../../assets/image/web/seat/senior-seat-selected-up.svg')
    center center no-repeat;
    background-size: 32px  32px;

  }
  .senior-seat{
    background-size: 32px  32px;

  }
  .add-seat{
    pointer-events: none;
    display:flex;
    justify-content:center;
    align-items:center;
    width:55px;height:56px;
    background: rgba(13, 175, 156, 0.15);
    border-radius:2px;position: absolute;
  }
  .add-seat-unset{
    pointer-events: none;
    display:flex;
    justify-content:center;
    align-items:center;
    width:55px;height:56px;
    background: rgba(253, 87, 81, 0.15);
    border-radius:2px;position: absolute;
  }
  .unenable-events{
      pointer-events: none;
  }
  .grid-content {
    width: 132px;
    height: 132px;
    border-radius: 2px;
    border: 1px solid #D2E0F3;
    text-align: center;
    div{
       margin-left: 34px;
       margin-top: 25px;
       margin-bottom: 15px;
       height: 50px;
       width: 62px;
    }
    img{
        margin-left: 43px;
       margin-top: 25px;
       margin-bottom: 15px;
       display: block;
    }
   #table{
       background: url('../../../assets/image/web/seat/table.svg')
       center center no-repeat;
      background-size: 60px 48px;
    }
    #mainScreen{
       background: url('../../../assets/image/web/seat/mainScreen.svg')
       center center no-repeat;
       background-size: 50px 48px;
      }
    #rostrum{
        background: url('../../../assets/image/web/seat/rostrum.svg')
        center center no-repeat;
       background-size: 56px 48px;
    }
    #door{
     background: url('../../../assets/image/web/seat/door.svg')
     center center no-repeat;
     background-size: 40px 48px;
   }
    &:hover{
      border: 1px solid @green;
      background: #F3FFFE;
      #table{
       background: url('../../../assets/image/web/seat/table-hover.svg')
       center center no-repeat;
            background-size: 60px 48px;
      }
      #mainScreen{
       background: url('../../../assets/image/web/seat/mainScreen-hover.svg')
      center center no-repeat;
      background-size: 50px 48px;
    }
      #rostrum{
        background: url('../../../assets/image/web/seat/rostrum-hover.svg')
        center center no-repeat;
                   background-size: 56px 48px;
      }
      #door{
       background: url('../../../assets/image/web/seat/door-hover.svg')
       center center no-repeat;
        background-size: 40px 48px;
      }
    }
  }
  .grid-content-disable {
    width: 132px;
    height: 132px;
    border-radius: 2px;
    border: 1px solid #e4e8eb;
    text-align: center;
    background: #f4f8f9;
    div {
      margin-left: 34px;
       margin-top: 25px;
       margin-bottom: 15px;
       height: 50px;
       width: 62px;
    }
   #rostrum{
      background: url('../../../assets/image/web/seat/rostrum-disable.svg')
      center center no-repeat;
      background-size: 56px 48px;
    }
  #door{
     background: url('../../../assets/image/web/seat/door-disable.svg')
     center center no-repeat;
    background-size: 40px 48px;
   }
  }

  .dialog-model{
    height: 252px;
    background: #F3FFFE;

  }
 .dialog-model-detail{
    height: 390px;
      .el-input {
      width: 100px;

    }
  }
  .div-preview{
    display: flex;
    justify-content:center;
    align-items:center;
    width: 197px;
    height: 198px;
    opacity: 0.8;
    border: 1px solid #dcfaf6;
    // background:-moz-linear-gradient(top,transparent 65px, #86D7CD 66px ),
    //-moz-linear-gradient(left, transparent 65px, #86D7CD 66px);
    background:-webkit-linear-gradient(top,transparent 65px, #dcfaf6 65px),
    -webkit-linear-gradient(left, transparent 65px, #dcfaf6 65px);
     background-color:rgba(244,255,254,0.42)!important;
    // background-color: #F4FFFE;
    background-size: 66px 66px;
  }
  .div-preview-ie{
     background:-ms-linear-gradient(top,transparent 65px, #dcfaf6 66px),
     -ms-linear-gradient(left, transparent 65px, #dcfaf6 66px);
    //  background-color: #F4FFFE;
    background-color:rgba(244,255,254,0.42)!important;
    background-size: 66px 66px;
  }
  .child-pre{
   position:  relative;
  }
  .tipsSpan{
      font-size: 10px;
      font-family: PingFangSC-Regular, PingFang SC;
      font-weight: 400;
      color: #B2C7E3;
      line-height: 14px;
  }

 .unselect {
    -webkit-user-select: none;
    -moz-user-select: none;
    -khtml-user-select: none;
    -ms-user-select: none;
    -o-user-select: none;
     user-select: none;
}
.block-area{
      //  opacity: 0.1;
      // background-color: red;
      position: absolute;
}
.popDiv{
  position: absolute;
  opacity:0.5;
  background-color:red;
 pointer-events: none;
}
</style>
