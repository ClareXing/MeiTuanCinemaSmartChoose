<template>
  <div class="wrapper">
    <div class='title-wrapper'>
        <el-input class="vue-input"
          maxlength="20"
          size="medium"
          suffix-icon="el-icon-edit"
          placeholder="请输入模板名称"
          v-model="seatModelInfo.modelName">
        </el-input>
    </div>
    <div class="btn-wrapper">
        <el-row :gutter="30">
             <el-col :span="2">  <el-button type="primary"  @click="selModel">选择模型</el-button></el-col>
             <el-col :span="2">  <el-button type="primary"  @click="addFacility">添加设施</el-button></el-col>
             <el-col :span="2">  <el-button  @click="addSeat" :type="!enableAddSeat?'primary' : ''">添加座位</el-button></el-col>
             <el-col :span="2"><el-button type="primary" icon="el-icon-delete" @click="clearAll">清空全部</el-button></el-col>
             <el-col :span="16" class="el-col_empty"></el-col>
             <!-- <el-col :span="3" >  <span>点击的类型</span></el-col>
             <el-col :span="2" > <el-button type="primary" icon="el-icon-edit" @click="edit">编辑</el-button></el-col>
             <el-col :span="2" >  <el-button type="primary" icon="el-icon-delete" @click="deleteSeat">删除</el-button></el-col>
            <el-col :span="1" class="el-col_empty"></el-col> -->
         </el-row>
     </div>
     <div class="seat-wrapper">
        <div class="inner-seat-wrapper" ref="innerSeatWrapperRef"  @click="click" @mousemove='mousemove' >
          <!--模型布局-->
          <div v-for="(row,rowIndex) in gridArray" :key="rowIndex">
                <div v-for="(item,colIndex) in row" :key="colIndex"
                  class="square-block" :style="{width:gridSize+'px',height:gridSize+'px'}">
                      <div v-if="gridArray[rowIndex][colIndex].type!==-1" class="inner-seat"
                       :class="gridArray[rowIndex][colIndex].type===1?'selected-seat':'unselected-seat'"
                       :row="rowIndex" :column="colIndex" >
                      </div>
                </div>
          </div>

         <div class='edit-form'>
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
                <el-form-item label="座位朝向" v-if="showSelFacilityOrientation">
                <!-- <el-input v-model="seatModelInfo.orientation"></el-input> -->
                    <el-select v-model="facilityInfo.facilityOrientation" placeholder="请选择">
                      <el-option
                        v-for="item in orientations"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                      </el-option>
                    </el-select>
                </el-form-item>
               </el-form>
            </div>
          </el-card>
         </div>

         <!--添加座位时的图片-->
          <div v-show="enableAddSeat" ref="addSeatDivRef" id='addSeatDivRef' class="add-seat">
            <img src="../assets/enableAddSeat.png" width="100%" height="100%"/>
          </div>

          <!--可移动状态座位模型鼠标点击定位时消失-->
          <seats @initSeatModelArray='initSeatModelArray' :type="seatModelInfo.seatModelType+'座位类型'"  ref="seatsRef"
                :gridSize="parseInt(gridSize)"
                :circleNum="parseInt(seatModelInfo.circleNum)"
                :seatModelArrayHeight="parseInt(seatModelArrayHeight)"
                :seatModelArrayWidth="parseInt(seatModelArrayWidth)"
                :modelType="seatModelInfo.seatModelType"
                :orientation="seatModelInfo.orientation" v-if="showDraggModel">
            </seats>
          <!--可移动设施-->
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
          <!--围墙-->
           <!-- <wall ></wall> -->
         </div>
        </div>

       <div class ='footer-wrapper'>
        <el-button type="primary" @click="cancel">取消</el-button>
        <el-button type="primary" @click="saveAndQuit">保存并退出</el-button>
       </div>

       <!--弹出的对话框区域-->
        <el-dialog title="添加模型" :visible.sync="dialogModelSelectVisible"
            :show-close="false" width="600px" :close-on-click-modal="false">
            <span >请选择需要添加的座位模型</span>
          <el-row :gutter="20">
            <el-col :span="6"><div class="grid-content bg-purple" > <img src="../assets/U.png" @click="nextModel('uShape')"/></div></el-col>
            <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/huixing.png"  @click="nextModel('gyrationShape')"/></div></el-col>
            <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/kezhuoxing.png"  @click="nextModel('deskShape')"/></div></el-col>
            <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/zidingyixing.png"  @click="nextModel('customize')"/></div></el-col>
          </el-row>
        </el-dialog>

        <el-dialog title="编辑模型" :visible.sync="dialogModelInfoVisible" width="600px" :close-on-click-modal="false">
          <el-form>
              <el-form-item label="U型方向" v-show="showSelOrientation">
                    <el-select v-model="seatModelInfo.orientation" placeholder="请选择">
                      <el-option
                        v-for="item in orientations"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                      </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="横向座位数">
                  <el-input-number v-model="seatModelInfo.horizontalSeatNum" :min="1" :max="18" label="横向座位数"></el-input-number>
                </el-form-item>
                <el-form-item label="纵向座位数">
                    <el-input-number v-model="seatModelInfo.verticalSeatNum" :min="1" :max="18" label="纵向座位数"></el-input-number>
                </el-form-item>
                <!-- <el-form-item label="环数">
                  <el-input-number v-model="seatModelInfo.circleNum" :min="1" :max="3" label="环数"></el-input-number>
                <!-- <el-input v-model="seatModelInfo.circleNum"></el-input> -->
              <!--</el-form-item> -->
          </el-form>
            <div slot="footer" class="dialog-footer">
              <el-button @click="upModel">上一步</el-button>
              <el-button type="primary" @click="okModel">确 定</el-button>
            </div>
        </el-dialog>

        <el-dialog title="选择设施" :visible.sync="dialogFacilityVisible"
          width="600px" :close-on-click-modal="false"  :show-close="false" >
          <el-row :gutter="20">
          <el-col :span="6"><div class="grid-content bg-purple" > <img src="../assets/Rostrum.png" @click="nextFacility('Rostrum')"></div></el-col>
          <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/table.png"  @click="nextFacility('table')"></div></el-col>
          <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/meetingScreen.png"  @click="nextFacility('meetingScreen')"></div></el-col>
          <el-col :span="6"><div class="grid-content bg-purple"> <img src="../assets/indoor.png"  @click="nextFacility('indoor')"></div></el-col>
        </el-row>
        </el-dialog>

        <el-dialog title="添加设施" :visible.sync="dialogFacilityDetailVisible" :close-on-click-modal="false" >
          <el-form>
            <el-row>
              <el-col :span=16>
            <el-form-item label="设施朝向" v-if="showSelFacilityOrientation">
              <!-- <el-input v-model="seatModelInfo.orientation"></el-input> -->
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
                <el-input-number v-model="facilityInfo.facilityWidth" :min="1" :max="18" label="设施长度"></el-input-number>
            </el-form-item>
              <el-form-item label="设施高度">
              <el-input-number v-model="facilityInfo.facilityHeight" :min="1" :max="18" label="设施高度"></el-input-number>
            </el-form-item>
            <el-form-item label="座位数量"  v-if="showSelFacilityOrientation">
              <el-input-number v-model="facilityInfo.holdSeatNum" :min="1" :max="18" label="设施高度"></el-input-number>
            </el-form-item>
           </el-col>
           <!--模型预览-->
            <el-col>
                  <seats
                gridSize="5"
                :circleNum="parseInt(seatModelInfo.circleNum)"
                :seatModelArrayHeight="parseInt(seatModelArrayHeight)"
                :seatModelArrayWidth="parseInt(seatModelArrayWidth)"
                :modelType="seatModelInfo.seatModelType"
                :orientation="seatModelInfo.orientation">
                   </seats>
            </el-col>
            </el-row>
          </el-form>

        <div slot="footer" class="dialog-footer">
          <el-button @click="upFacility">上一步</el-button>
          <el-button type="primary" @click="okFacility">确 定</el-button>
        </div>
        </el-dialog>
       <!--编辑座位时候的遮罩层-->
        <div :style="maskStyle" class="mask" @click="enableAddSeat = false" v-if="enableAddSeat"></div>

  </div>
</template>
<script>
   import facility from './customComponents/facility.vue'
   import wall from './customComponents/wall.vue'
   import seats from  './customComponents/seats.vue'
   export default {
    name: 'seatLayout',
    components:{
      facility,
      wall,
      seats
    },
    // 监听属性
    watch: {
      gridArray(value){
         console.log(value)
      }
    },
   // 计算属性 围墙的位置
    computed: {
        // 根据座位数计算模型座位区域宽度
        seatModelArrayWidth:function(){
              //如果类型是'uShape' 或者'gyrationShape
              if(this.seatModelInfo.seatModelType==='uShape'||this.seatModelInfo.seatModelType==='gyrationShape'){
                return this.seatModelInfo.horizontalSeatNum+2
              }else{
                return this.seatModelInfo.horizontalSeatNum
              }
        },
         // 根据座位数计算模型座位区域高度
       seatModelArrayHeight:function(){
            //如果类型是'uShape'
           if(this.seatModelInfo.seatModelType==='uShape'){
               return  this.seatModelInfo.verticalSeatNum+1
            }else if (this.seatModelInfo.seatModelType==='gyrationShape'){ // 如果类型是 ,'gyrationShape
              return  this.seatModelInfo.verticalSeatNum+2
            }
            else{
              return  this.seatModelInfo.verticalSeatNum
            }
        },
        // top:function(){
        //    for(item in this.gridArray){
        //      for(j in item){
        //        if(item[j]==0)
        //      }
        //    }
        // },
        // bottom:function(){

        // },
        // left:function(){

        // },
        // right:function(){

        // }
    },
		data () {
			return {
        // 遮罩层的样式
        maskStyle: null,
        // 座位编辑div 显示
        editDivStyle:null,
        // 是否可以添加座位
        enableAddSeat:false,
        // 座位模型是否可编辑
        enableEdit:false,
        // 座位设施是否可删除
        enableDelete:false,
        // 存储布局的格子二维数组
        gridArray:[],
        // 布局行数
        rows:20,
        // 布局列数
        colums:24,
        // 布局格子的大小
        gridSize:0,
        // 模型选择对话框显示
        dialogModelSelectVisible :true,
        //模型编辑对话框显示
        dialogModelInfoVisible :false,
        //添加设施对话框显示
        dialogFacilityVisible:false ,
        //设施对话框显示
        dialogFacilityDetailVisible:false,
        //编辑座位设施详细信息
        showEditInfo:false,
        //模型是否显示朝向选择
        showSelOrientation:false,
        //设施朝向设置是否显示
        showSelFacilityOrientation:false,
        // 设置设施座位数量显示
        showHoldSeatNum:false,
        // 可移动状态的模型显示
        showDraggModel:false,
        // 可移动状态的设施
        showDraggFacility:false,
       // 添加座位模型时记录的座位模型信息
       seatModelInfo : {
         modelName:'', // 模型名称
         seatModelType:'',//座位模型('uShape','gyrationShape','deskShape,'customize')
         verticalSeatNum: 7 ,//纵向座位数
         horizontalSeatNum: 7,//横向座位数:
         orientation: 'u',//朝向（上下左右 一次 u d l r）默认朝上,u型以外的模型默认为空
         circleNum: 1, // 有多少环，最多支持三环
       },
       //座位模型数组（临时的，当点击确定时将值赋给gridArray）
       seatModelArray:[],
       //添加设施时记录的设施信息
       facilityInfo:{
           id:0,//设施id（用来区分相同设施，便于编辑删除）
           topIndex:2 ,// 设施顶部位于布局的第几格
           leftIndex:2 ,// 设施左边部分位于布局的第几格
           facilityWidth:2, //设施横向占多少格
           facilityHeight:2,//设施纵向占多少格
           facilityType: '',   // 设施图片名称
           holdSeatNum:2,// 设施容纳的座位数量
           facilityOrientation:'u' //设施朝（上下左右 依次 u d l r）默认朝上
        },
        // 座位模板信息
        seatTemplate:{
          /*
          {
            二维数组中每个元素存储以下信息
            type：'普通座位':1,高管座：2 残疾人座3 '设施':4,设施+座位8 围墙:9
            id: 设施存设施id，座位存参会人
            lable：编号
          }
           */
          layoutArray:[],//布局的二维数组
          facilities:[],// 设施信息
          templateName:'',//模板名称
          wallWidth:0, // 围栏的长
          wallHeight:0,   // 围栏的高
          wallStartLeft:0,  // 围栏Left位置
          wallStartTop:0,  // 围栏Top位置
        },

       // 围墙
          wall:{
            topIndex:0,
            leftIndex:0,
            wallWidth:0,//围墙长度
            wallHeight:0,//围墙宽度
          },
          seatType:'',
           //座位类型
        seatTypes: [{
            value:1,
            label:'普通座'
          },{
              value:2,
            label:'高管座'
          },{
              value:3,
              label:'残疾人座'
          }],
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
      deleteSeat:function(){
        this.enableEdit = false
        this.enableDelete = false
      },
      //删除座位模型
      deleteSeatsModel:function(index){
        if (index !== 0) {
            this.seatsModel.splice(index, 1)
        }
      },
      //删除设施
      deleteFacilitiy:function(index){
         if(index!==0){
            this.seatsModel.splice(index,1)
         }
      },
      // 计算围墙
      computeWall:function(){

      },
      // 模型选择
      nextModel:function(type){
          // 选择模型 的对话框关闭
          this.dialogModelSelectVisible= false
          this.seatModelInfo.seatModelType = type
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
        //关闭对话框
        this.showDraggModel= true
        this.dialogModelInfoVisible=false
      },

      // 返回设施选择
      upFacility:function(){
          this.dialogFacilityVisible =true
          this.dialogFacilityDetailVisible=false
      },
      // 确定添加设施
      okFacility:function(){
         //new 一个设施
         //关闭设施设置对话框
        this.dialogFacilityDetailVisible=false
        this.showDraggFacility=true

      },
       // 设施选择
      nextFacility:function(type){
           // 选择设施 的对话框关闭
          this.dialogFacilityVisible= false
          // this.seatModelInfo.seatModelType = type
          this.facilityInfo.facilityType = type+'.png'
          // U 型 需要设置朝向
          if(type==='Rostrum'){
            this.showHoldSeatNum=true;
            this.showSelFacilityOrientation=true;
          }else{
            this.showHoldSeatNum=false;
            this.showSelFacilityOrientation=false;
          }
          //弹出 设施参数设置对话框
           this.dialogFacilityDetailVisible=true;
      },

      // 打开模型选择
      selModel:function(){
        // this.resetSeat()
        this.dialogModelSelectVisible= true

      },
      // 添加设施
      addFacility:function(){
         this.dialogFacilityVisible= true
         this.dialogFacilityDetailVisible=false
      },
      // 添加单个座位
      addSeat:function(){
        this.enableAddSeat = true
      },
      //
      handleSeatClick:function(){

      },
       // 清空全部
      clearAll:function(){
          this.$confirm('此操作将清空当前画布全部内容，是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            //点击确定的操作  清空画布
            this. resetSeat();
            this.facilities=[];
          }).catch(() => {
            //点取消的提示
          });
          this.enableAddSeat= false

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
        let oldArray = this.gridArray.slice();
        for(let i=0;i<this.rows;i++){
          for(let j=0;j<this.colums;j++){
              oldArray[i][j]=0
          }
        }
        this.gridArray = oldArray;
      },
      //取消模型
      cancel:function(){

      },
      //保存并退出
      saveAndQuit:function(){

      },
      // 初始化添加座位模型时模型数组
      initSeatModelArray:function(arr){
         this.seatModelArray = arr;
      },
      //鼠标事件
      //鼠标移动事件
     mousemove(e) {
       // 座位模型
      if(this.$refs.seatsRef){
          let width = (this.seatModelArrayWidth* this.gridSize)
          let height = (this.seatModelArrayHeight * this.gridSize)
          let left = e.target.offsetLeft-10-height/2
          let top = e.target.offsetTop -width/2
          let style = `left: ${left}px;top: ${top}px;width:${width+10*this.seatModelArrayWidth}px;height:${height+10*this.seatModelArrayHeight}px`
          this.$refs.seatsRef.$el.style=style
        }
        // 设施
        if(this.$refs.facilityRef){
          let width = (this.facilityInfo.facilityWidth* this.gridSize)
          let height = (this.facilityInfo.facilityHeight* this.gridSize)
          let left = e.target.offsetLeft-10-height/2
          let top = e.target.offsetTop -width/2
          let style = `left: ${left}px;top: ${top}px;width:${width+10*(this.seatModelInfo.horizontalSeatNum-1)}px;height:${height+10*(this.seatModelInfo.verticalSeatNum-1)}px`
          this.$refs.facilityRef.$el.style=style
        }
        //添加单个座位
        if(this.enableAddSeat){
            let row = e.target.getAttribute('row')
            if (!row) {
                return
            }
            let top = e.target.offsetTop
             let left = e.target.offsetLeft
            let style = `left: ${left}px;top: ${top}px;width:${this.gridSize}px;height:${this.gridSize};background:red;position: absolute;`
            this.$refs.addSeatDivRef.style=style;
        }
      },
       //鼠标按下事件
      click(e) {
             //判断点击的位置是否在格子上
            let row = e.target.getAttribute('row')
            if (!row) {
                return
            }
            let column = e.target.getAttribute('column')
            //如果有可以拖动的设备或者模型点击后 确定位置取消拖拽状态
            if(this.showDraggModel){
              //计算开始位置
              let topIndex= row-Math.floor(this.seatModelArrayWidth/2)
              let leftIndex= column-Math.floor(this.seatModelArrayHeight/2)
              console.log(this.seatModelArrayWidth,this.seatModelArrayHeight)
              console.log(this.seatModelArray)
              //将座位的数据添加到二维数组中
              //step 1 判断放置的位置是否被占
              for(let i=0;i<this.seatModelArrayHeight;i++){
                for(let j=0;j<this.seatModelArrayWidth;j++){
                   if( this.gridArray[i+topIndex][j+leftIndex].type!==0 && this.seatModelArray[i][j]!==0){
                         this.showDraggModel= false
                       return ;
                   }
                  }
              }
              // setp2 放置座位
              for(let i=0;i<this.seatModelArrayHeight;i++){
                for(let j=0;j<this.seatModelArrayWidth;j++){
                  // 只修改没有座位的位置
                  if(this.seatModelArray[i][j]!==0){
                    this.gridArray[i+topIndex][j+leftIndex]= {'type':this.seatModelArray[i][j],'id':'','label':''};
                  }
                }
              }
              this.showDraggModel= false
            }else if(this.showDraggFacility){  // 如果添加设施
              //计算开始位置
              let topIndex= row-Math.floor(this.facilityInfo.facilityWidth/2)
              let leftIndex= column-Math.floor(this.facilityInfo.facilityHeight/2)
              //将座位的数据添加到二维数组中
              for(let i=0;i<this.facilityInfo.facilityWidth;i++){
              for(let j=0;j<this.facilityInfo.facilityHeight;j++){
                  this.gridArray[i+topIndex][j+leftIndex]={'type':4,'id':topIndex.toString()+leftIndex.toString(),'label':''} //设施占位标志（如果有必要 可以根据不同的设施设置不同的值
                }
              }
              this.showDraggFacility=false
            }else if(this.enableAddSeat){ // 如果是添加座位
                // 判断点击的位置是否被占
                if (this.gridArray[row][column].type!==0){
                  return;
                }
               this.$set(this.gridArray[row], column, {'type':1,'id':'','label':''})
            }else {//编辑座位或者其它

                if (this.gridArray[row][column].type===0){
                  return;
                }else if(this.gridArray[row][column].type===1){
                  //显示出编辑框
                  this.showEditInfo=true;
                }

            }
       },

      //初始座位数组
      initSeatArray: function(){
        //初始化布局
        let gridArray = Array(this.rows).fill({'type':0,'id':'','label':''}).map(()=>Array(this.colums).fill({'type':0,'id':'','lable':''}));
        this.gridArray = gridArray;
        // 计算每个格子的尺寸  （画布的长度-（横向格子数+1）*（margin-left+margin-right））/横向格子数
        this.gridSize=parseInt(window.getComputedStyle(this.$refs.innerSeatWrapperRef).width)/this.colums-10;
      },
    },

    mounted() {
      this.initSeatArray();
      // console.log(window.screen.width)
       this.maskStyle = `{ height: ${document.body.clientHeight}px; height: ${document.body.clientHeight}px;}`
      // console.log(this.style)
    },

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
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
  .wrapper{
    height:100%;
    padding: 20px;
    box-sizing: border-box;
  }
  .title-wrapper{
    width:1221px;
    margin: 0px auto;
    height:40px;
  }

.vue-input /deep/ .el-input__inner {
    border: 1px solid #fff !important;
    }
    .vue-input /deep/.el-input{
    width: 280px;
    }

.btn-wrapper{
    width:1221px;
    margin: 0px auto;
    height:45px;
    text-align: center;
    background:#EBEBEB;
    padding-top: 5px;
    padding-left: 5px;
  }
 .footer-wrapper{
    width:1221px;
    margin: 0px auto;
    height:45px;
    text-align: right;
    background:#EBEBEB;
    padding-top: 5px;
    padding-right: 15px;
 }
  .seat-wrapper{
    width:1221px;
    height:1020px;
    border:1px dashed #c5c5c5;
    margin: 0 auto;
    position: relative;
    overflow: hidden;
  }
  /*画布区域大小*/
  .inner-seat-wrapper{
    height:1000px;
    width:1201px;
    top:10px;
    position: relative;
    bottom:0px;
    box-sizing: content-box;
    padding: 0px 10px;     /* overflow :hidden; */
    z-index: 1000;
  }
  /*画布中小正方形 */
  .square-block{
    float:left;
    display: flex;
    justify-content: center;
    align-items: center;
    border: white 5px solid;
    background: #EBEBEB;

  }
  .el-col_empty{
    min-height:1px;
  }

    .inner-seat{
    width:100%;
    height:100%;
    cursor: pointer;
  }
 .selected-seat{
    background: url('../assets/selected.png') center center no-repeat;
    background-size: 100% 100%;
        line-height: 42px;
  }
  .unselected-seat{
    background-size: 100% 100%;
  }
  .add-seat{
          pointer-events: none;
  }
  .edit-form {
    pointer-events: none;
    position: absolute;
  }

</style>
