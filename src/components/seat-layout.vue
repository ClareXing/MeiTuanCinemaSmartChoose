<template>
 <div class="pr seatLayout" ref='seatLayoutRef'>
     <div class="clearfix">
            <el-input class="vue-input"
          maxlength="20"
          size="medium"
          suffix-icon="el-icon-edit"
          placeholder="请输入模板名称"
          v-model="seatTemplate.name">
        </el-input>
        <hr style="border:0;height:2px;background: #F5F6FB" />
      </div>
     <div class="btn-wrapper">
       <el-button type="primary"  @click="selModel">选择模型</el-button>
       <el-button type="primary"  @click="addFacility">添加设施</el-button>
       <el-button  @click="addSeat" :type="!enableAddSeat?'primary' : ''">添加座位</el-button>
       <el-button type="primary" icon="el-icon-delete" @click="clearAll">清空全部</el-button>
     </div>
     <div class="seat-wrapper" ref="seatWrapperRef">
        <div class="inner-seat-wrapper"  @click="click"
        @mousemove.stop='mousemove'  ref="innerSeatWrapperRef">
          <!--模型布局-->
          <div v-for="(row,rowIndex) in seatTemplate.layoutArray" :key="rowIndex"
          style="display: flex;">
            <div v-for="(item,colIndex) in row" :key="colIndex"
              :class="seatTemplate.layoutArray[rowIndex][colIndex].type!== 0?
              'square-noneblock':'square-block'"
              :style="{width:gridSize+'px',height:gridSize+'px'}">
              <div :class="seatStyle(seatTemplate.layoutArray[rowIndex][colIndex].type,
              seatTemplate.layoutArray[rowIndex][colIndex].orientation)"
                :row="rowIndex" :column="colIndex" class ='normal-seat'>
                {{seatTemplate.layoutArray[rowIndex][colIndex].type}}
                {{seatTemplate.layoutArray[rowIndex][colIndex].orientation}}
              </div>
            </div>
          </div>
          <!--设施-->
          <div v-for="(facility,index) in seatTemplate.facilities"
          :key="index+'20000'" @click=editFacility(facility)
              :class="enableEventclass"
              :style="{width:facility.facilityWidth*(gridSize+doubbleBorder)+'px',
                  height:facility.facilityHeight*(gridSize+doubbleBorder)+'px',
                  top:(gridSize+doubbleBorder)*facility.topIndex+'px',
                  left:(gridSize+doubbleBorder)*facility.leftIndex+'px'}" class="div-facility">
              <img :src="require('../assets/image/web/seat/'+facility.facilityType+'.svg')"
              width="100%" height= "100%" :class="enableEventclass">
          </div>
           <!--围墙  -->
         <wall :borderWidth="gridSize"  ref="wallRef"
                :wallHeight="seatTemplate.wallHeight"
                  :wallWidth="seatTemplate.wallWidth"
                  :startLeftIndex="seatTemplate.wallStartLeft"
                  :startTopIndex="seatTemplate.wallStartTop"
                :rectArr="seatTemplate.layoutArray" v-if='!editState&&seatTemplate.wallWidth>0'>
                <slot name ='door'>
                  <span>门</span>
                </slot>
        </wall>
        <!--座位或设施编辑框-->
         <div class='edit-form' v-if='showEditForm' :style="editDivStyle">
           <el-card class="box-card">
            <div slot="header" class="clearfix">
              <span>{选择的类型}</span>
              <el-button style="float: right; padding: 3px 0" type="text">操作按钮</el-button>
            </div>
            <div  class=" item">
               <el-form>
                   <el-form-item label="座位类型">
                    <el-select v-model="seatType" placeholder="请选择座位类型">
                      <el-option
                        v-for="item in seatTypes"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                      </el-option>
                    </el-select>
                   </el-form-item>
                   <el-form-item label="座位朝向" >
                    <el-select v-model="facilityInfo.facilityOrientation" placeholder="请选择">
                      <el-option
                        v-for="item in orientations"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                      </el-option>
                    </el-select>
                   </el-form-item>
                   <el-form-item label="编辑设施">
                   </el-form-item>
               </el-form>
            </div>
          </el-card>
         </div>

         <!--添加单个座位时的图片-->
          <div v-show="enableAddSeat" ref="addSeatDivRef" id='addSeatDivRef' class="add-seat">
            <img src="../assets/image/web/seat/normal-seat-down.svg"
            width="100%" height="100%"/>
          </div>

          <!--可移动状态座位模型鼠标点击定位时消失-->
          <seats @initSeatModelArray='initSeatModelArray'
               :type="seatModelInfo.seatModelType+'座位类型'"  ref="seatsRef"
                :gridSize="parseInt(gridSize)"
                :circleNum="parseInt(seatModelInfo.circleNum)"
                :seatModelArrayHeight="parseInt(seatModelArrayHeight)"
                :seatModelArrayWidth="parseInt(seatModelArrayWidth)"
                :modelType="seatModelInfo.seatModelType"
                :orientation="seatModelInfo.orientation" v-if="showDraggModel">
          </seats>
          <!--可移动设施时显示-->
          <facility :type="'设施'" ref="facilityRef"
                :gridSize="parseInt(gridSize)"
                :topIndex="parseInt(facilityInfo.topIndex)"
                :leftIndex="parseInt(facilityInfo.leftIndex)"
                :facilityWidth="parseInt(facilityInfo.facilityWidth)"
                :facilityHeight="parseInt(facilityInfo.facilityHeight)"
                :facilityType="facilityInfo.facilityType"
                :holdSeatNum="parseInt(facilityInfo.holdSeatNum)"
                :facilityOrientation="facilityInfo.facilityOrientation"
                v-if="showDraggFacility">
          </facility>

         </div>
     </div>

       <!--弹出的对话框区域-->
        <el-dialog title="添加模型" :visible.sync="dialogModelSelectVisible"
          :show-close="false" width="694px" custom-class='dialog-model'
          :close-on-click-modal="false">
          <el-row :gutter="20">
            <el-col :span="6">
              <div class="grid-content"  @click="nextModel('uShape')">
               <img src="../assets/image/web/seat/uShape.svg" />
               <span >U型排布</span>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content" @click="nextModel('gyrationShape')">
                 <img src="../assets/image/web/seat/gyrationShape.svg"  />
                 <span >环型排布</span>
              </div>
            </el-col>
            <el-col :span="6">
              <div class="grid-content" @click="nextModel('deskShape')">
              <img src="../assets/image/web/seat/deskShape.svg"  />
              <span >课桌式排布</span>
              </div>
          </el-col>
            <el-col :span="6">
              <div class="grid-content" @click="nextModel('customize')">
               <img src="../assets/image/web/seat/customize.svg"  />
               <span >自定义排布</span>
             </div>
            </el-col>
          </el-row>
        </el-dialog>

        <el-dialog title="编辑模型" :visible.sync="dialogModelInfoVisible"
        width="652px"  custom-class='dialog-model-detail'
        :close-on-click-modal="false">
            <el-row>
               <el-col :span="14">
                  <el-form  label-position="right">
                   <el-form-item label="U开口方向" v-show="seatModelInfo.seatModelType==='uShape'" >
                        <el-select v-model="seatModelInfo.orientation" placeholder="请选择">
                          <el-option
                            v-for="item in orientations"
                            :key="item.value"
                            :label="item.label"
                            :value="item.value">
                          </el-option>
                        </el-select>
                    </el-form-item>

                    <el-form-item
                    :label="seatModelInfo.seatModelType==='uShape'?'u底部座位':'横向座位数'">
                      <el-input-number   v-model="seatModelInfo.horizontalSeatNum"
                       controls-position="right" :min="1" :max="13"></el-input-number>
                    </el-form-item>

                    <el-form-item
                    :label="this.seatModelInfo.seatModelType==='uShape'?'u侧边座位':'纵向座位数'">
                        <el-input-number v-model="seatModelInfo.verticalSeatNum"
                          controls-position="right"
                        :min="1" :max="13" label="纵向座位数"></el-input-number>
                    </el-form-item>

              <!--模型预览-->
              </el-form>
              </el-col>
                <el-col :span="10">
                  <div style="text-align:center;" class='div-preview'>
                      <seats  v-model="seatModelInfo"
                      :gridSize="4"
                      :circleNum="parseInt(seatModelInfo.circleNum)"
                      :seatModelArrayHeight="parseInt(seatModelArrayHeight)"
                      :seatModelArrayWidth="parseInt(seatModelArrayWidth)"
                      :modelType="seatModelInfo.seatModelType"
                      :orientation="seatModelInfo.orientation"
                      :reqType="2">
                     </seats>
                  </div>
              </el-col>
            </el-row>
            <div slot="footer" class="dialog-footer">
              <el-button @click="upModel">上一步</el-button>
              <el-button type="primary" @click="okModel">确 定</el-button>
            </div>
        </el-dialog>

        <el-dialog title="选择设施" :visible.sync="dialogFacilityVisible"
          width="694px" :close-on-click-modal="false"
          :show-close="false" custom-class='dialog-model'>
          <el-row :gutter="20">
          <el-col :span="6">
            <div class="grid-content"  @click="nextFacility('rostrum')">
              <img src="../assets/image/web/seat/rostrum.svg">
              <span>主席台</span>
            </div>
          </el-col>
          <el-col :span="6">
            <div class="grid-content"  @click="nextFacility('door')">
               <img src="../assets/image/web/seat/door.svg" >
                <span>入口</span>
            </div>
         </el-col>
          <el-col :span="6">
          <div class="grid-content" @click="nextFacility('table')">
            <img src="../assets/image/web/seat/table.svg"  >
             <span>会议桌</span>
          </div>
        </el-col>
          <el-col :span="6"><div class="grid-content" @click="nextFacility('mainScreen')">
             <img src="../assets/image/web/seat/mainScreen.svg"  >
              <span>主屏幕</span>
           </div>
          </el-col>

        </el-row>
        </el-dialog>

        <el-dialog title="添加设施" :visible.sync="dialogFacilityDetailVisible"
        :close-on-click-modal="false"
         width="652px"  custom-class='dialog-model-detail'>
           <el-row>
               <el-col :span="16">
          <el-form>
            <el-form-item label="设施朝向" v-if="facilityInfo.facilityType!=='table'">
                <el-select v-model="facilityInfo.facilityOrientation" placeholder="请选择">
                  <el-option
                    v-for="item in orientations"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
                  </el-option>
            </el-select>
            </el-form-item>
            <el-form-item >
            </el-form-item>
              <el-form-item label="设施长度" label-width="120px"
              v-if="facilityInfo.facilityType!=='door'">
                <el-input-number v-model="facilityInfo.facilityWidth"
                :min="1" :max="18" label="设施长度"  controls-position="right"></el-input-number>
            </el-form-item>
              <el-form-item label="设施高度" label-width="120px"
               v-if="facilityInfo.facilityType==='rostrum'
                    ||facilityInfo.facilityType==='table'">
              <el-input-number v-model="facilityInfo.facilityHeight"
               :min="1" :max="18"  controls-position="right"></el-input-number>
            </el-form-item>
            <el-form-item label="座位数量" label-width="120px"
            v-if="facilityInfo.facilityType==='rostrum'">
              <el-input-number v-model="facilityInfo.holdSeatNum"
              :min="1" :max="18"  controls-position="right"></el-input-number>
            </el-form-item>
          </el-form>
               </el-col>
               <el-col :span="8">
                 <div style="text-align:center;" class='div-preview'>
                  <img :src="imageSrc"/>
                 </div>
               </el-col>
           </el-row>

        <div slot="footer" class="dialog-footer">
          <el-button @click="upFacility">上一步</el-button>
          <el-button type="primary" @click="okFacility">确 定</el-button>
        </div>
        </el-dialog>
       <!--编辑座位时候的遮罩层-->
        <div :style="maskStyle" class="mask" @click="endEditState" v-if="editState"></div>


  <div class="tc mb0_item pb34 pt50" style="background: #F5F6FB;margin-bottom:0;">
      <el-button type="primary" class="w100" plain @click="cancel">取消</el-button>
      <el-button type="primary" class="w100" @click="saveAndQuit">保存并退出</el-button>
    </div>
  </div>
</template>
<script>
import facility from './component/facility.vue';
import wall from './component/wall.vue';
import seats from './component/seats.vue';

export default {
  components: {
    facility,
    wall,
    seats,
  },
  // 监听属性
  watch: {
    // seatTemplate.layoutArray(value){
    // }
  },
  // 计算属性 围墙的位置
  computed: {
    // 根据座位数计算模型座位区域宽度
    seatModelArrayWidth() {
      // 如果类型是'uShape' 或者'gyrationShape
      if (this.seatModelInfo.seatModelType === 'uShape'
           || this.seatModelInfo.seatModelType === 'gyrationShape') {
        return this.seatModelInfo.horizontalSeatNum + 2;
      }
      return this.seatModelInfo.horizontalSeatNum;
    },
    // 根据座位数计算模型座位区域高度
    seatModelArrayHeight() {
      // 如果类型是'uShape'
      if (this.seatModelInfo.seatModelType === 'uShape') {
        return this.seatModelInfo.verticalSeatNum + 1;
      } if (this.seatModelInfo.seatModelType === 'gyrationShape') { // 如果类型是 ,'gyrationShape
        return this.seatModelInfo.verticalSeatNum + 2;
      }
      return this.seatModelInfo.verticalSeatNum;
    },

    enableEventclass() {
      return this.enableEvent ? '' : 'unenable-events';
    },

    // 是否编辑状态
    editState() {
      if (this.enableAddSeat || this.showDraggModel
      || this.showDraggFacility || this.showEditForm) {
        return true;
      }
      return false;
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
      // 遮罩层的样式
      maskStyle: null,
      // 座位编辑div 显示
      editDivStyle: null,
      // 网格的左+右边框宽度
      doubbleBorder: 10,
      // 是否可以添加座位
      enableAddSeat: false,
      // 设施鼠标事件可用状态
      enableEvent: true,
      // 布局行数
      rows: 20,
      // 布局列数
      colums: 24,
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
        verticalSeatNum: 7, // 纵向座位数
        horizontalSeatNum: 7, // 横向座位数:
        orientation: 'u', // 朝向（上下左右 一次 u d l r）默认朝上,u型以外的模型默认为空
        circleNum: 1, // 有多少环，最多支持三环
      },
      // 座位模型数组（临时的，当点击确定时将值赋给gridArray）
      seatModelArray: [],
      // 添加设施时记录的当前要添加的设施信息
      facilityInfo: {
        id: 0, // 设施id（用来区分相同设施，便于编辑删除）
        topIndex: 2, // 设施顶部位于布局的第几格
        leftIndex: 2, // 设施左边部分位于布局的第几格
        facilityWidth: 2, // 设施横向占多少格
        facilityHeight: 2, // 设施纵向占多少格
        facilityType: '', // 设施图片名称
        holdSeatNum: 2, // 设施容纳的座位数量
        facilityOrientation: 'u', // 设施朝（上下左右 依次 u d l r）默认朝上
      },
      imageSrc: '', // 选择的设施模型图片
      // 座位模板信息
      seatTemplate: {
        /*
          {
            二维数组中每个元素存储以下信息
            type：'普通座位':1,高管座：2 '设施':4,设施+座位8 围墙:9
            orientation：座位或者设施的朝向
            id: 设施存设施id，座位存参会人（没有为空）
            lable：编号
          }
        */
        layoutArray: [], // 布局的二维数组
        facilities: [], // 设施信息
        name: '', // 模板名称
        wallWidth: 0, // 围栏的长
        wallHeight: 0, // 围栏的高
        wallStartLeft: 0, // 围栏Left位置
        wallStartTop: 0, // 围栏Top位置
      },
      // 编辑座位类型
      seatType: '',
      // 座位类型
      seatTypes: [{
        value: 1,
        label: '普通座',
      }, {
        value: 2,
        label: '高管座',
      }],
      // 朝向
      orientations: [{
        value: 'u',
        label: '向上',
      }, {
        value: 'd',
        label: '向下',
      }, {
        value: 'l',
        label: '向左',
      }, {
        value: 'r',
        label: '向右',
      }],
      value: '',
    };
  },

  methods: {
    // 计算围墙
    computeWall() {
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
        this.seatTemplate.wallWidth = right - left + 3;// 围墙长度
        this.seatTemplate.wallHeight = bottom - top + 3;// 围墙高度
        this.seatTemplate.wallStartTop = top - 2;//
        this.seatTemplate.wallStartLeft = left - 2;
      }
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
    },
    // 返回模型选择
    upModel() {
      this.dialogModelInfoVisible = false;
      this.dialogModelSelectVisible = true;
    },
    // 确定模型
    okModel() {
      // 关闭对话框
      this.showDraggModel = true;
      this.dialogModelInfoVisible = false;
      // 添加模型时 禁用 设施的鼠标事件（确定位置后开启）
      this.enableEvent = false;
    },

    // 返回设施选择
    upFacility() {
      this.dialogFacilityVisible = true;
      this.dialogFacilityDetailVisible = false;
    },
    // 确定添加设施
    okFacility() {
      // new 一个设施
      // 关闭设施设置对话框
      this.dialogFacilityDetailVisible = false;
      this.showDraggFacility = true;
    },
    // 设施选择
    nextFacility(type) {
      // 选择设施 的对话框关闭
      this.dialogFacilityVisible = false;
      // this.seatModelInfo.seatModelType = type
      this.facilityInfo.facilityType = type;
      // 除了会议桌不需要设置朝向

      this.imageSrc = `../assets/image/web/seat/${this.facilityInfo.facilityType}.svg`;

      // 弹出 设施参数设置对话框
      this.dialogFacilityDetailVisible = true;
    },

    // 打开模型选择
    selModel() {
      this.dialogModelSelectVisible = true;
    },
    // 添加设施
    addFacility() {
      this.dialogFacilityVisible = true;
      this.dialogFacilityDetailVisible = false;
    },
    // 添加单个座位
    addSeat() {
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
    // 重置设施属性
    resetFacility() {
      this.facilityInfo.topIndex = 0;
      this.facilityInfo.leftIndex = 0;
      this.facilityInfo.facilityWidth = 0;
      this.facilityInfo.facilityHeight = 0;
      this.facilityInfo.facilityType = '';
      this.facilityInfo.holdSeatNum = 0;
      this.facilityInfo.facilityOrientation = 'u';
    },
    // 重置座位
    resetSeat() {
      // 将所有座位的值变为0
      const oldArray = this.seatTemplate.layoutArray.slice();
      for (let i = 0; i < this.rows; i += 1) {
        for (let j = 0; j < this.colums; j += 1) {
          oldArray[i][j] = {
            type: 0, id: '', label: '', orientation: '',
          };
        }
      }
      this.seatTemplate.layoutArray = oldArray;
      this.seatTemplate.wallHeight = 0;
      this.seatTemplate.wallWidth = 0;
    },
    // 点击编辑座位
    editFacility(fac) {
      // 显示出编辑框
      // if(fac.facilityType==='table'){

      // }
      const left = (fac.leftIndex + fac.facilityWidth / 2) * this.gridSize;
      const top = (fac.topIndex + fac.facilityHeight) * this.gridSize;
      const style = `left: ${left}px;top: ${top}px;width:400px;height:200px`;
      this.editDivStyle = style;
      this.showEditForm = true;
    },
    // 结束编辑状态
    endEditState() {
      this.enableAddSeat = false;
      this.showDraggModel = false;
      this.showDraggFacility = false;
      this.showEditForm = false;
    },
    // 取消模型
    cancel() {
      this.$router.go(-1);
    },
    // 保存并退出
    saveAndQuit() {

    },

    // 初始化添加座位模型时模型数组
    initSeatModelArray(arr) {
      this.seatModelArray = arr;
    },
    // 鼠标事件
    // 鼠标移动事件
    mousemove(e) {
      // 座位模型
      if (this.$refs.seatsRef) {
        const newWith = this.seatModelArrayWidth * (this.gridSize + this.doubbleBorder);
        const newHeight = this.seatModelArrayHeight * (this.gridSize + this.doubbleBorder);

        const left = e.clientX - this.$refs.seatWrapperRef.offsetLeft - Math.ceil(newWith / 2);
        const top = e.clientY - this.$refs.seatWrapperRef.offsetTop - Math.ceil(newHeight / 2);


        const style = `left: ${left}px;top: ${top}px;width:${newWith}px;height:${newHeight}px`;
        this.$refs.seatsRef.$el.style = style;
      } else if (this.$refs.facilityRef) { // 移动状态的设施
        const newWith = this.facilityInfo.facilityWidth * (this.gridSize + this.doubbleBorder);
        const newHeight = this.facilityInfo.facilityHeight * (this.gridSize + this.doubbleBorder);
        // console.log(e.clientX)
        // console.log(this.$refs.seatWrapperRef.offsetLeft)
        const left = e.clientX - this.$refs.seatWrapperRef.offsetLeft - Math.ceil(newWith / 2);
        const top = e.clientY - this.$refs.seatWrapperRef.offsetTop - Math.ceil(newHeight / 2);

        const style = `left: ${left}px;top: ${top}px;width:${newWith}px;height:${newHeight}px`;
        this.$refs.facilityRef.$el.style = style;
      } else if (this.enableAddSeat) { // 添加单个座位
        const row = e.target.getAttribute('row');
        if (!row) {
          return;
        }
        const top = e.target.offsetTop;
        const left = e.target.offsetLeft;
        const style = `left: ${left}px;top: ${top}px;width:${this.gridSize}px;height:${this.gridSize};background:red;position: absolute;`;
        this.$refs.addSeatDivRef.style = style;
      }
    },
    // 鼠标点击事件
    click(e) {
      // 判断点击的位置是否在格子上
      const row = e.target.getAttribute('row');
      if (!row) {
        return;
      }
      const column = e.target.getAttribute('column');
      // 如果有可以拖动的设施或者模型点击后 确定位置取消拖拽状态
      if (this.showDraggModel) {
        // 计算开始位置
        const topIndex = row - Math.ceil(this.seatModelArrayHeight / 2) + 1;
        const leftIndex = column - Math.ceil(this.seatModelArrayWidth / 2) + 1;

        // 将座位的数据添加到二维数组中
        // step 1 判断放置的位置是否被占
        for (let i = 0; i < this.seatModelArrayHeight; i += 1) {
          for (let j = 0; j < this.seatModelArrayWidth; j += 1) {
            if (this.seatTemplate.layoutArray[i + topIndex][j + leftIndex].type !== 0
            && this.seatModelArray[i][j].type !== 0) {
              this.showDraggModel = false;
              this.enableEvent = true;// 开启设施图片的鼠标事件
              return;
            }
          }
        }
        // setp 2 放置座位
        for (let i = 0; i < this.seatModelArrayHeight; i += 1) {
          for (let j = 0; j < this.seatModelArrayWidth; j += 1) {
            // 只修改没有座位的位置
            if (this.seatModelArray[i][j].type !== 0) {
              this.seatTemplate.layoutArray[i + topIndex][j + leftIndex] = {
                type: this.seatModelArray[i][j].type, id: '', label: '', orientation: this.seatModelArray[i][j].orientation,
              };
            }
          }
        }
        // step 3 计算围墙
        this.computeWall();
        // step 4 座位模型添加成功移除拖拽的 模型图片
        this.showDraggModel = false;
        this.enableEvent = true;// 开启设施图片的鼠标事件
      } else if (this.showDraggFacility) { // 如果添加设施
        // 计算开始位置
        const topIndex = row - Math.ceil(this.facilityInfo.facilityHeight / 2) + 1;
        const leftIndex = column - Math.ceil(this.facilityInfo.facilityWidth / 2) + 1;

        // step 1 判断放置的位置是否被占
        for (let i = 0; i < this.facilityInfo.facilityHeight; i += 1) {
          for (let j = 0; j < this.facilityInfo.facilityWidth; j += 1) {
            if (this.seatTemplate.layoutArray[i + topIndex][j + leftIndex].type !== 0) {
              this.showDraggModel = false;
              return;
            }
          }
        }
        // step2 将设备信息添加到 设备数组中
        const facilityTmp = {
          id: topIndex.toString() + leftIndex.toString(),
          topIndex,
          leftIndex,
          facilityWidth: this.facilityInfo.facilityWidth,
          facilityHeight: this.facilityInfo.facilityHeight,
          facilityType: this.facilityInfo.facilityType,
        };
        this.seatTemplate.facilities.push(facilityTmp);

        // step3 将设备的占位信息添加到 布局二维数组中
        for (let i = 0; i < this.facilityInfo.facilityHeight; i += 1) {
          for (let j = 0; j < this.facilityInfo.facilityWidth; j += 1) {
            this.seatTemplate.layoutArray[i + topIndex][j + leftIndex] = {
              type: 4,
              id: topIndex.toString() + leftIndex.toString(),
              label: '',
              orientation: this.facilityInfo.facilityOrientation,
            }; // 设施占位标志（如果有必要 可以根据不同的设施设置不同的值
          }
        }

        // step 4 计算围墙(添加座位时 围墙与座位之间空一圈)
        this.computeWall();
        // 设备添加成功 移除拖拽的设备图片
        this.showDraggFacility = false;
      } else if (this.enableAddSeat) { // 如果是添加座位
        // 判断点击的位置是否被占
        if (this.seatTemplate.layoutArray[row][column].type !== 0) {
          return;
        }
        this.$set(this.seatTemplate.layoutArray[row], column, {
          type: 1, id: '', label: '', orientation: '',
        });
        // step 3 计算围墙
        this.computeWall();
      } else { // 编辑单个座位
        const tempSeatType = this.seatTemplate.layoutArray[row][column].type;
        // 如果点击的位置没有东西
        if (tempSeatType === 0) {

          // 1 普通座位,2 高管座
        } else if (tempSeatType === 1 || tempSeatType === 2) {
          // 显示出编辑框
          const left = e.target.offsetLeft;
          const top = e.target.offsetTop;
          const style = `left: ${left}px;top: ${top}px;width:200px;height:200px`;
          this.editDivStyle = style;

          this.showEditForm = true;
        }
      }
    },

    // 初始座位数组
    initSeatArray() {
      // 初始化布局
      const gridArray = Array(this.rows).fill({
        type: 0, id: '', label: '', orientation: '',
      })
        .map(() => Array(this.colums).fill({
          type: 0, id: '', lable: '', orientation: '',
        }));
      this.seatTemplate.layoutArray = gridArray;
      // 计算每个格子的尺寸  （画布的长度-（横向格子数+1）*（margin-left+margin-right））/横向格子数
      this.gridSize = parseInt(
        window.getComputedStyle(this.$refs.innerSeatWrapperRef).width, 10,
      ) / this.colums - this.doubbleBorder;
    },
  },

  mounted() {
    // 初始化 布局二维数组
    this.initSeatArray();
    // 初始化遮罩层大小
    this.maskStyle = `{ height: ${document.body.clientHeight}px; height: ${document.body.clientHeight}px;}`;
  },

};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.vue-input /deep/ .el-input__inner {
    border: 1px solid #fff !important;
    }
    .vue-input /deep/.el-input{
      width: 280px;
    }


   /* 编辑操作时的 背景框 */
  .mask {
    width: 100%;
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
  .title-wrapper{
    width:1221px;
    margin: 0px auto;
    height:40px;
  }

.btn-wrapper{
    width:1221px;
    margin: 0px auto;
    height:45px;
    text-align: left;
    padding-left: 15px;
    margin-bottom: 20px;

  }
   .btn-wrapper  button{
      width: 117px;
      height: 36px;
      margin-right: 10px;
      background: #0DAF9C;
      border-radius: 2px;
      border: 1px solid #0DAF9C;
    }

  .seat-wrapper{
    height: 1020px;
    width:1221px;
    border: 1px solid #0DAF9C;
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
    width:1201px;
    top:10px;
    position: relative;
    bottom:0px;
     box-sizing: content-box;
    left:10px;
    z-index: 1000;
    background: #FCFCFC;
    border-radius: 2px;
    border: 1px solid #E4E8EB;
  }
  /*画布中小正方形 */
  .square-block{
    /* float:left; */
    border: white 5px solid;
    background: #EBEBEB;
  }
 .square-noneblock{
    border: white 5px solid;
    background: white;
 }
  .el-col_empty{
    min-height:1px;
  }

.inner-seat{
    width:100%;
    height:100%;
    cursor: pointer;
  }
 .normal-seat-left{
     background: url('../assets/image/web/seat/normal-seat-left.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .normal-seat-right{
    background: url('../assets/image/web/seat/normal-seat-right.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
   .normal-seat-up{
     background: url('../assets/image/web/seat/normal-seat-up.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .normal-seat-down{
    background: url('../assets/image/web/seat/normal-seat-down.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
   .senior-seat-left{
     background: url('../assets/image/web/seat/senior-seat-left.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
 .senior-seat-right{
    background: url('../assets/image/web/seat/senior-seat-right.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
   .senior-seat-down{
     background: url('../assets/image/web/seat/senior-seat-down.svg') center center no-repeat;
    background-size: 100% 100%;
   line-height: 42px;
  }
     .senior-seat-up{
     background: url('../assets/image/web/seat/senior-seat-up.svg') center center no-repeat;
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
    pointer-events: none;
    position: absolute;
  }
  .div-facility{
    position:absolute;
    opacity: 1;
  }
  .unenable-events{
      pointer-events: none;
  }

  .grid-content {
    width: 132px;
    height: 132px;
    border-radius: 2px;
    border: 1px solid #D2E0F3;

      display: grid;
    vertical-align: middle;
    text-align: center;


    /* &:hover{
      background: #F3FFFE;
    } */
  }
    .grid-content   img {
          margin-left: 43px;
    margin-top: 25px;
      display: block;
    }

  .dialog-model{
    height: 252px;
    background: #FFFFFF;

  }
 .dialog-model-detail{
    height: 390px;
  }
  .div-preview{
    width: 197px;
    height: 198px;
    background: #F4FFFE;
    opacity: 0.42;
    border: 1px solid #86D7CD;
  }

</style>
