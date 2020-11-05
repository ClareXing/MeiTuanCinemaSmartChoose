<template>
  <div class="wrapper">
    <div class="seat-wrapper">
      <div >
      <div class="btn-buy" @click="selModel">
        选择模型
      </div>
      <div class="btn-buy" @click="addFacility">
        添加设施
    </div>
    <div class="btn-buy" @click="addFacility">
        添加座位
    </div>
    <div class="group-btn">
    <el-button-group>
      <el-button type="primary" icon="el-icon-edit" @click="enableEdit"></el-button>
      <el-button type="primary" icon="el-icon-delete" @click="enableDelete"></el-button>
    </el-button-group>
    </div>
  </div>
      <div class="inner-seat-wrapper" ref="innerSeatWrapper">
        <div v-for="row in seatRow" :key="row"   >
          <!--这里的v-if很重要，如果没有则会导致报错，因为seatArray初始状态为空-->
          <div v-for="col in seatCol" :key="col" v-if="seatArray.length>0"
            class="square-block" :style="{width:seatSize+'px',height:seatSize+'px'}">
            <seat :v-if="seatArray[row-1][col-1]!==0"
            :seatType='seatArray[row-1][col-1].toString()'
            seatOrientation='u' seatLable="张三"/>
          </div>
        </div>
        <!--座位-->
        <!-- <seat :v-if="seatArray[row-1][col-1]!==0" :seatType='seatArray[row-1][col-1].toString()' seatOrientation='u' seatLable="张三"/> -->

       <!--设施-->
        <facility/>
        <!--div围墙-->
        <div id='divBlock'></div>
      </div>
    </div>
    <el-dialog title="选择模型" :visible.sync="dialogModelSelectVisible"
        :show-close="false" width="600px" :close-on-click-modal="false">
      <el-row :gutter="20">
        <el-col :span="6"><div class="grid-content bg-purple" > <img src="../assets/U.png" @click="nextModel('uShape')"/></div></el-col>
        <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/huixing.png"  @click="nextModel('gyrationShape')"/></div></el-col>
        <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/kezhuoxing.png"  @click="nextModel('deskShape')"/></div></el-col>
        <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/zidingyixing.png"  @click="nextModel('customize')"/></div></el-col>
      </el-row>
    </el-dialog>

    <el-dialog title="编辑模型" :visible.sync="dialogModelInfoVisible" width="600px">
      <el-form>
      <el-form-item label="U型方向" v-show="showSelOrientation">
        <!-- <el-input v-model="seatModelConfig.orientation"></el-input> -->
          <el-select v-model="seatModelConfig.orientation" placeholder="请选择">
            <el-option
              v-for="item in orientations"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
      </el-select>
      </el-form-item>
      <el-form-item label="横向座位数">
          <el-input v-model="seatModelConfig.seatRowNum"></el-input>
      </el-form-item>
        <el-form-item label="纵向座位数">
            <el-input v-model="seatModelConfig.seatColNum"></el-input>
      </el-form-item>
        <!-- <el-form-item label="环数">
          <el-input-number v-model="seatModelConfig.circleNum" :min="1" :max="3" label="环数"></el-input-number>
        <!-- <el-input v-model="seatModelConfig.circleNum"></el-input> -->
      <!--</el-form-item> -->
      </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="upModel">上一步</el-button>
      <el-button type="primary" @click="okModel">确 定</el-button>
    </div>
    </el-dialog>
    <el-dialog title="选择设施" :visible.sync="dialogFacilityVisible"
      width="600px">
      <el-row :gutter="20">
      <el-col :span="6"><div class="grid-content bg-purple" > <img src="../assets/Rostrum.png" @click="nextFacility('rostrum')"></div></el-col>
      <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/table.png"  @click="nextFacility('meetingTable')"></div></el-col>
      <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/meetingScreen.png"  @click="nextFacility('meetingScreen')"></div></el-col>
      <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/indoor.png"  @click="nextFacility('door')"></div></el-col>
    </el-row>
    </el-dialog>
    <el-dialog title="添加设施" :visible.sync="dialogFacilityDetailVisible">
      <el-form>
      <el-form-item label="设施朝向" v-if="showSelFacilityOrientation">
        <!-- <el-input v-model="seatModelConfig.orientation"></el-input> -->
          <el-select v-model="facilityInfo.facilityOrientation" placeholder="请选择">
            <el-option
              v-for="item in orientations"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
      </el-select>
      </el-form-item>
      <el-form-item label="设施大小">
      </el-form-item>
        <el-form-item label="设施长度">
          <el-input v-model="facilityInfo.facilityWidth"></el-input>
      </el-form-item>
        <el-form-item label="设施高度">
            <el-input v-model="facilityInfo.facilityHeight"></el-input>
      </el-form-item>

      <el-form-item label="座位数量"  v-if="showSelFacilityOrientation">
         <el-input v-model="facilityInfo.holdSeatNum"></el-input>
      </el-form-item>
      </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="upFacility">上一步</el-button>
      <el-button type="primary" @click="okFacility">确 定</el-button>
    </div>
    </el-dialog>

  </div>
</template>
<script>
   import seat from './customComponents/seat.vue'
   import facility from './customComponents/facility.vue'
	export default {
    name: 'seatLayout',
    components:{
      seat,
      facility
    },
    // 监听属性
    watch: {
      firstName: function (val) {
        this.fullName = val + ' ' + this.lastName;
      },
      lastName: function (val) {
        this.fullName = this.firstName + ' ' + val;
      },
    },
   // 计算属性
    computed: {
          reversedMessage: function () {
            // `this` 指向 vm 实例
            return this.message.split('').reverse().join('');
      },
    },
		data () {
			return {
        // 座位是否可编辑
        enableEdit:false,
        // 座位是否可删除
        enableDelete:false,
        // 存储布局的格子二维数组
        seatArray:[],
        // 布局行数
        seatRow:20,
        // 布局列数
        seatCol:24,
        // 布局格子的大小
        seatSize:0,
        // 模型选择对话框显示
        dialogModelSelectVisible :true,
        //模型编辑对话框显示
        dialogModelInfoVisible :false,
        //添加设施对话框显示
        dialogFacilityVisible:false ,
        //设施对话框显示
        dialogFacilityDetailVisible:false,
        //模型是否显示朝向选择
        showSelOrientation:false,
        //设施朝向设置是否显示
        showSelFacilityOrientation:false,
        // 设置设施座位数量显示
        showHoldSeatNum:false,
       // 座位模板信息
       seatModelConfig : {
         modelName:'', // 模板名称
         seatModelType:'',//座位模型('uShape','gyrationShape','deskShape,'customize')
         seatColNum: 7 ,//横向座位数
         seatRowNum: 7,//纵向座位数:
         orientation: 'u',//朝向（上下左右 一次 u d l r）默认朝上,u型以外的模型默认为空
         circleNum: 1, // 有多少环，最多支持三环
         facilitys:[],//设备信息 facilityInfo
         seats:[],// 座位信息
       },
       // 设施信息
       facilityInfo:{
           topIndex:0 ,// 设施顶部位于布局的第几格
           leftIndex:0 ,// 设施左边部分位于布局的第几格
           facilityWidth:0, //设施横向占多少格
           facilityHeight:0,//设施纵向占多少格
           facilityType: '',   // 设施名称 （主屏幕 mainscreen）
           holdSeatNum:0,// 设施容纳的座位数量
           facilityOrientation:'u' //设施朝（上下左右 依次 u d l r）默认朝上
        },
      //   // 座位信息
      //   seatInfo:{
      //      seatType:'',//座位类型（普通座位，主席座等）
      //      rowIndex:0, //座位位于布局的第几行
      //      colIndex:0, //座位位于布局的第几列
      //      seatNo:'', //座位编号
      //      seatLable:'' ,//座位文字显示信息（与座位号二者只能显示其中一个）
      //      seatOrientation:''//座位朝向（上下左右 依次 u d l r）默认朝上
      //   },
       // 朝向
       orientations: [{
                value: 'u',
                label: '向上'
              }, {
                value: 'd',
                label: '向下'
              }, {
                value: 'l',
                label: '向左'
              }, {
                value: 'r',
                label: '向右'
              }],
              value: ''
			}
		},
    methods:{
      // 座位编辑
      edit:function(){
        this.enableEdit =true
        this.enableDelete =false
      },
      // 座位删除
      delete:function(){
        this.enableEdit = false
        this.enableDelete = true
      },
      // 模型选择
      nextModel:function(type){
          // 选择模型 的对话框关闭
          this.dialogModelSelectVisible= false
          this.seatModelConfig.seatModelType = type
          // U 型 需要设置朝向
          if(type==='uShape'){
            this.showSelOrientation=true;
          }else{
              this.showSelOrientation=false;
          }
          // 不是自定义类型要弹出设置模型参数对话框
          if(type!=='customize'){
          this.dialogModelInfoVisible =true
          }
      },
      // 返回模型选择
      upModel:function(){
          this.dialogModelInfoVisible =false
          this.dialogModelSelectVisible=true
      },
      //确定模型
      okModel:function(){
        //渲染座位
        this.renderSeat();
        //关闭对话框
        this.dialogModelInfoVisible=false
      },
      // 返回设施选择
      upFacility:function(){
          this.dialogFacilityVisible =true
          this.dialogFacilityDetailVisible=false
      },
      okFacility:function(){
         //new 一个设施
         //关闭设施设置对话框
        this.dialogFacilityDetailVisible=false

      },
       // 设施选择
      nextFacility:function(type){
           // 选择设施 的对话框关闭
          this.dialogFacilityVisible= false
          this.seatModelConfig.seatModelType = type
          // U 型 需要设置朝向
          if(type==='rostrum'){
            this.showHoldSeatNum=true;
            this.showSelFacilityOrientation=true;
          }else{
            this.showHoldSeatNum=false;
            this.showSelFacilityOrientation=false;
          }
          //弹出 设施参数设置对话框
           this.dialogFacilityDetailVisible=true;
      },

      //打开模型选择
      selModel:function(){
        this.resetSeat()
        this.dialogModelSelectVisible= true

      },
      //添加设施
      addFacility:function(){
         this.resetFacility();
         this.dialogFacilityVisible= true
         this.dialogFacilityDetailVisible=false
      },

      //重置设施属性
      resetFacility:function(){
        this.facilityInfo.topIndex=0;
        this.facilityInfo.leftIndex=0;
        this.facilityInfo.facilityWidth=0;
        this.facilityInfo.facilityHeight=0;
        this.facilityInfo.facilityType='';
        this.facilityInfo.holdSeatNum=0;
        this.facilityInfo.facilityOrientation='u';
      },
    	//重置座位
      resetSeat: function(){
        //将所有座位的值变为0
        let oldArray = this.seatArray.slice();
        for(let i=0;i<this.seatRow;i++){
          for(let j=0;j<this.seatCol;j++){
              oldArray[i][j]=0
          }
        }
        this.seatArray = oldArray;
      },
      //渲染座位区域 目前只有U型
      renderSeat:function(){
        let seatColNum = parseInt(this.seatModelConfig.seatColNum);
        let seatRowNum = parseInt(this.seatModelConfig.seatRowNum);
        let orientation = this.seatModelConfig.orientation;
        let circleNum =parseInt(this.seatModelConfig.circleNum);
        //确定第一个环的左上开始位置(最外环为第一个环)
        let  startRowIndex = Math.ceil(this.seatRow/2)-Math.ceil(seatRowNum /2)  // 根据布局大小计算环开始的行位置
        let startColIndex = Math.ceil(this.seatCol/2)-Math.ceil(seatColNum /2)  // 根据布局大小计算环开始的列位置

        let oldArray = this.seatArray.slice();
        oldArray=this.hLayout(seatRowNum+4,seatColNum+4,startRowIndex-2,startColIndex-2,oldArray,2);

        // validate(); //校验逻辑
        //如果是U型
        //朝向上或者下
        //环数1 则 横向位数 不能小于3 纵向不能小于2  且都不能大于20
        //环数2 则 横向位数 不能小于5 纵向不能小于3  且都不能大于20
        //环数3 则 横向位数 不能小于7 纵向不能小于4  且都不能大于20
        //朝向左或者右
        //环数1 则 横向位数 不能小于2 纵向不能小于3  且都不能大于20
        //环数2 则 横向位数 不能小于3 纵向不能小于5  且都不能大于20
        //环数3 则 横向位数 不能小于4 纵向不能小于7  且都不能大于20
        //校验逻辑也可以不写在这里直接 用rule 规则取校验

        // 目前只考虑一个环
        if( this.seatModelConfig.seatModelType==='uShape'){
        if(circleNum===1){
            //画环
            oldArray=this.hLayout(seatRowNum,seatColNum,startRowIndex,startColIndex,oldArray);
            //根据方向去掉多余的座位
            this.seatArray= this.uLayout(startRowIndex,startColIndex,seatColNum,seatRowNum,oldArray,orientation)
        }else if(circleNum===2){
            // 画第一个环
            oldArray=this.hLayout(seatRowNum,seatColNum,startRowIndex,startColIndex,oldArray);
            //画第二个环
            oldArray= this.hLayout(seatRowNum-2,seatColNum-2,startRowIndex+1,startColIndex+1,oldArray);
            // 去掉第一个环多余座位
            oldArray= this.uLayout(startRowIndex,startColIndex,seatColNum,seatRowNum,oldArray,orientation)
            // 去掉第二个环多余座位
            this.seatArray= this.uLayout(startRowIndex+1,startColIndex+1,seatColNum-2,seatRowNum-2,oldArray,orientation,2)
        }else if(circleNum===3){
            // 画第一个环
            oldArray=this.hLayout(seatRowNum,seatColNum,startRowIndex,startColIndex,oldArray);
            //画第二个环
            oldArray= this.hLayout(seatRowNum-2,seatColNum-2,startRowIndex+1,startColIndex+1,oldArray);
            // 画第三个环
            oldArray= this.hLayout(seatRowNum-4,seatColNum-4,startRowIndex+2,startColIndex+2,oldArray);
            // 去掉第一个环多余座位
            oldArray= this.uLayout(startRowIndex,startColIndex,seatColNum,seatRowNum,oldArray,orientation)
            // 去掉第二个环多余座位
            oldArray= this.uLayout(startRowIndex+1,startColIndex+1,seatColNum-2,seatRowNum-2,oldArray,orientation,2)
            // 去掉第三个环多余座位
            this.seatArray= this.uLayout(startRowIndex+2,startColIndex+2,seatColNum-4,seatRowNum-4,oldArray,orientation,3)
        }else{
              alert('请设置环数小于等于三的正整数')
       }
        }else if( this.seatModelConfig.seatModelType==='gyrationShape'){
          //回型
            this.seatArray=this.hLayout(seatRowNum,seatColNum,startRowIndex,startColIndex,oldArray);
        }else if(this.seatModelConfig.seatModelType==='deskShape'){
           //课桌型
           this.seatArray=this.tableLayout(seatRowNum,seatColNum,startRowIndex,startColIndex,oldArray);
        }else{

        }
       console.log(this.seatArray);
    },

      /** 根据朝向 去掉环中多余的座位
       * seatRowNum:横向座位数
       * seatColNum:纵向座位数
       * startRowIndex:左上角开始行位置
       * startColIndex:左上角开始的列位置
       * oldArray:需要标注画环的二维数组
       * orientation:朝向
       * circleNum:当前处理第几个环（从外到里依次1、2、3）
       */
      uLayout:function(startRowIndex,startColIndex,seatColNum,seatRowNum,oldArray,orientation,circleNum=1){
         //向上
         if(orientation==='u'){
            for(let i=1;i<seatColNum-1;i++){
                oldArray[startRowIndex][startColIndex+i] = 0
            }
            oldArray[startRowIndex+seatRowNum-1][startColIndex]=0
            oldArray[startRowIndex+seatRowNum-1][startColIndex+seatColNum-1]=0
            if(circleNum===2){
              oldArray[startRowIndex-1][startColIndex] = 1
              oldArray[startRowIndex-1][startColIndex+seatColNum-1] = 1
            }else if(circleNum===3){
              oldArray[startRowIndex-1][startColIndex] = 1
              oldArray[startRowIndex-1][startColIndex+seatColNum-1] = 1
              oldArray[startRowIndex-2][startColIndex] = 1
              oldArray[startRowIndex-2][startColIndex+seatColNum-1] = 1
            }
            return oldArray;
         }else if(orientation==='d'){
            //向下
            for(let i=1;i<seatColNum-1;i++){
              oldArray[startRowIndex+seatRowNum-1][startColIndex+i] = 0
            }
            oldArray[startRowIndex][startColIndex]=0
            oldArray[startRowIndex][startColIndex+seatColNum-1]=0

             if(circleNum===2){
              oldArray[startRowIndex+seatRowNum][startColIndex] = 1
              oldArray[startRowIndex+seatRowNum][startColIndex+seatColNum-1] = 1
            }else if(circleNum===3){
              oldArray[startRowIndex+seatRowNum][startColIndex] = 1
              oldArray[startRowIndex+seatRowNum][startColIndex+seatColNum-1] = 1
              oldArray[startRowIndex+seatRowNum+1][startColIndex] = 1
              oldArray[startRowIndex+seatRowNum+1][startColIndex+seatColNum-1] = 1
            }
            return oldArray;
         }else if(orientation==='l'){
            //向左
            for(let i=1;i<seatRowNum-1;i++){
                oldArray[startRowIndex+i][startColIndex] = 0
            }
            oldArray[startRowIndex][startColIndex+seatColNum-1]=0
            oldArray[startRowIndex+seatRowNum-1][startColIndex+seatColNum-1]=0

            if(circleNum===2){
              oldArray[startRowIndex][startColIndex-1] = 1
              oldArray[startRowIndex+seatRowNum-1][startColIndex-1] = 1
            }else if(circleNum===3){
              oldArray[startRowIndex][startColIndex-1] = 1
              oldArray[startRowIndex+seatRowNum-1][startColIndex-1] = 1
              oldArray[startRowIndex][startColIndex-2] = 1
              oldArray[startRowIndex+seatRowNum-1][startColIndex-2] = 1
            }
            return oldArray;
         }else if(orientation==='r'){
            //向右
            for(let i=1;i<seatRowNum-1;i++){
                oldArray[startRowIndex+i][startColIndex+seatColNum-1] = 0
            }
            oldArray[startRowIndex][startColIndex]=0
            oldArray[startRowIndex+seatRowNum-1][startColIndex]=0

             if(circleNum===2){
              oldArray[startRowIndex][startColIndex+seatColNum] = 1
              oldArray[startRowIndex+seatRowNum-1][startColIndex+seatColNum] = 1
            }else if(circleNum===3){
              oldArray[startRowIndex][startColIndex+seatColNum] = 1
              oldArray[startRowIndex+seatRowNum-1][startColIndex+seatColNum] = 1
              oldArray[startRowIndex][startColIndex+seatColNum+1] = 1
              oldArray[startRowIndex+seatRowNum-1][startColIndex+seatColNum+1] = 1
            }
            return oldArray;
         }else{
           console.error('选择正确的朝向');
         }

      },

      /**
       * 根据横向和纵向的座位数画环
       * seatRowNum:横向座位数
       * seatColNum:纵向座位数
       * startRowIndex:左上角开始行位置
       * startColIndex:左上角开始的列位置
       * oldArray:需要标注画环的二维数组
       * setValue:设置环位置的数组坐标占位符
       */
      hLayout:function(seatRowNum,seatColNum,startRowIndex,startColIndex,oldArray,setValue=1){
           for(let i=0;i<seatRowNum-1;i++){
                oldArray[startRowIndex+i][startColIndex] = setValue
                oldArray[startRowIndex+i][startColIndex+seatColNum-1] = setValue
            }
          for(let j=0;j<seatColNum;j++){
               oldArray[startRowIndex+seatRowNum-1][startColIndex+j] = setValue
               oldArray[startRowIndex][startColIndex+j] = setValue
          }
           return oldArray;
      },
        /**
       * 根据横向和纵向的座位数布局课桌型
       * seatRowNum:横向座位数
       * seatColNum:纵向座位数
       * startRowIndex:左上角开始行位置
       * startColIndex:左上角开始的列位置
       * oldArray:需要标注画环的二维数组
       * setValue:设置环位置的数组坐标占位符
       */
      tableLayout:function(seatRowNum,seatColNum,startRowIndex,startColIndex,oldArray,setValue=1){
        for(let i=0;i<seatRowNum;i++){
          for(let j=0;j<seatColNum;j++){
                    oldArray[startRowIndex+i][startColIndex+j] = setValue
           }
        }
       return oldArray;
      },

      //初始座位数组
      initSeatArray: function(){
        //初始化布局
        let seatArray = Array(this.seatRow).fill(0).map(()=>Array(this.seatCol).fill(0));
        this.seatArray = seatArray;
        // 计算每个格子的尺寸  （画布的长度-（横向格子数+1）*（margin-left+margin-right））/横向格子数
        this.seatSize=(parseInt(window.getComputedStyle(this.$refs.innerSeatWrapper).width)-(this.seatCol+1)*10)/this.seatCol;
      },

  //开始拖拽事件
      onStart(){
        this.drag=true;
      },
      //拖拽结束事件
       onEnd() {
       this.drag=false;
      },

    },

    mounted:function(){
      this.initSeatArray(this.seatRow,this.seatCol)
    },


}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .wrapper{
    height:100%;
    padding: 40px;
    box-sizing: border-box;
  }
  .title{
    text-align: center;
  }
  .seat-wrapper{
    height:1400PX;
    width:1200px;
    border:1px dotted #c5c5c5;
    margin: 0 auto;
    position: relative;
    overflow:hidden;
  }
  .screen{
    margin: 0 auto;
    height:30px;
    width:300px;
    background-color: #e3e3e3;
    border-radius: 0 0 30px 30px;
    color: #585858;
    line-height: 30px;
    text-align: center;
  }
  .inner-seat-wrapper{
    position: absolute;
    top:120px;
    bottom:0;
    width:100%;
    box-sizing: border-box;
  }
  .inner-seat{
    width:100%;
    height:100%;
    cursor: pointer;
  }
  .square-block{
    float:left;
    display: flex;
    justify-content: center;
    align-items: center;
    /* margin: 5px 5px; */
    border: white 5px solid;
    background: #EBEBEB;
  }
  .screen-center{
    position: absolute;
    left:50%;
    transform: translateX(-50%);
    padding:5px;
    font-size: 13px;
    border-radius: 5px;
    top:50px;
    background-color: #f6f6f6;
    color: #636363;
    border:1px solid #b1b1b1;
  }
  .mid-line{
    position: absolute;
    left:50%;
    transform: translateX(-50%);
    top:100%;
    width:1px;
    height:800px;
    border-left:1px dashed #919191;
  }
  .btn-wrapper{
    margin: 20px auto;
    width:1000px;
    height:30px;
    text-align: center;
  }
  .btn-buy{
    height:100%;
    line-height: 30px;
    font-size: 14px;
    border-radius: 5px;
    padding:0 5px;
    background-color: #ffa349;
    color: #ffffff;
    display: inline-block;
    cursor: pointer;
    margin-right: 10px;
  }
  .group-btn{
    float: right;
  }
  .bought-seat{
     background: black;
    background-size: 100% 100%;
  }

</style>
