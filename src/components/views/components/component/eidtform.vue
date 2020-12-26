<template>
  <div :style="{width:width+'px',height:height+'px'}" class="div-edit">
               <el-form label-position="left">
                   <el-form-item label="座位类型" v-if="editInfo.type.indexOf('座位')!==-1"
                   label-width="92px">
                    <el-select v-model="seatTypeValue" placeholder="请选择座位类型">
                      <el-option
                        v-for="item in seatTypes"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                      </el-option>
                    </el-select>
                   </el-form-item>
                   <el-form-item label="座位方向"
                    v-if="editInfo.type.indexOf('座位')!==-1"
                    label-width="92px">
                    <el-select v-model="seatOrientationValue" placeholder="请选择座位方向">
                      <el-option
                        v-for="item in orientations"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                      </el-option>
                    </el-select>
                   </el-form-item>
                   <el-form-item label="编辑设施"  v-if="editInfo.type.indexOf('座位')===-1"
                    label-width="92px">
                   <el-link type="primary" :underline="true" @click="okEdit"
                    >编辑{{editInfo.type}}</el-link>
                   </el-form-item>
                     <el-form-item>
                       <div class="edit-btn-wrapper">
                      <el-button  icon="el-icon-delete" @click="deleteItems"
                      class='delete-btn'>删除</el-button>
                              <el-button type="primary" @click="okEdit"
                      v-if="editInfo.type.indexOf('座位')!==-1">确定</el-button>
                      <el-button @click="cancelEdit" type="primary"  plain>取消</el-button>
                       </div>
                    </el-form-item>
               </el-form>
            </div>
      <!-- </el-card>
         </div> -->
</template>
<script>

export default {
  props: ['editInfo', 'width', 'height', 'seatOrientation', 'seatType', 'refreshValue'],

  watch: {
    refreshValue() {
      this.seatOrientationValue = this.seatOrientation;
      this.seatTypeValue = this.seatType;
    },
  },

  data() {
    return {
      orientations: [{
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
      }],
      seatOrientationValue: this.seatOrientation,
      // 座位类型
      seatTypes: [{
        value: 1,
        label: '普通座',
      }, {
        value: 2,
        label: '高管座',
      }],
      seatTypeValue: this.seatType,
    };
  },
  methods: {
    deleteItems() {
      this.$emit('deleteItems');
    },
    okEdit() {
      this.$emit('okEdit', this.seatOrientationValue, this.seatTypeValue);
    },
    cancelEdit() {
      this.$emit('cancelEdit');
    },
  },

};
</script>

<style lang="less" scoped>

.div-edit{
  margin-top: 20px;
}

 .edit-btn-wrapper{
   button{
      width: 64px;
      height: 28px;
      border-radius: 2px;
      font-size: 14px;
      font-family: PingFangSC-Regular, PingFang SC;
      font-weight: 400;
      line-height: 4px;
      float:right
    }
    .delete-btn{
      float: left;
          color: #F56C6C;
          background: #fff;
          border: none;
          width:70px;
      }
 }
</style>
