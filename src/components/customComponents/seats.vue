<template>
<div :style="{width:seatModelArrayWidth*gridSize,height:seatModelArrayHeight*gridSize,
     top:gridSize*topIndex+'px',left:gridSize*leftIndex+'px'}" class="drag" >
     <!--根据相关参数画模型-->
    <div v-for="(row,rowIndex) in rectArray" :key="rowIndex">
      <div v-for="(item,colIndex) in row" :key="colIndex"
        class="square-block" :style="{width:gridSize+'px',height:gridSize+'px'}">
            <div v-if="rectArray[rowIndex][colIndex]!==-1" class="inner-seat"
              :class="rectArray[rowIndex][colIndex]===1?'selected-seat':'unselected-seat'">
            </div>
      </div>
    </div>
     <!--根据相关seats 复原座位-->
     <!--画小图-->
</div>
</template>

<script>
export default {
  /** 画模型 */
  props:{
      gridSize:Number,//单元格大小
      seatModelArrayWidth:Number,//宽
      seatModelArrayHeight:Number,//高
      modelType:String,//模型类型
      orientation:String,//朝向
      circleNum:Number,//环数
      topIndex:{
        type:Number,
        default:2
      },
      leftIndex:{
        type:Number,
        default:2
      },
      seatType:{
        type:Number,
        default:1
      },
      seatsArray:{type:Array,default:Array},//座位数组
      reqType:{
        type:Number,
        default:1,
      } //请求类型 （1画模型，2画预览）
  },
  watch:{
    // draggable:function(oldValue,newValue){
    //     if(!newValue){

    //     }
    // }
  },

  data(){
    return {
      rectArray:[],
      emptyFlag:0,
    }
  },
   methods:{
      //渲染座位区域 目前只有U型
      renderSeat:function(){

        let columns = this.seatModelArrayWidth
        let rows = this.seatModelArrayHeight
        let orientation = this.orientation
        let circleNum =this.circleNum;
        //确定第一个环的左上开始位置(最外环为第一个环)
        let  startRowIndex = 0  // 根据布局大小计算环开始的行位置
        let startColIndex = 0  // 根据布局大小计算环开始的列位置

        let oldArray = this.rectArray.slice();
        //oldArray=this.hLayout(rows+4,columns+4,startRowIndex-2,startColIndex-2,oldArray,2);
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
        if( this.modelType==='uShape'){
        if(circleNum===1){
            //画环
            oldArray=this.hLayout(rows,columns,startRowIndex,startColIndex,oldArray);
            //根据方向去掉多余的座位
            this.rectArray= this.uLayout(startRowIndex,startColIndex,columns,rows,oldArray,orientation)
        }else if(circleNum===2){
            // 画第一个环
            oldArray=this.hLayout(rows,columns,startRowIndex,startColIndex,oldArray);
            //画第二个环
            oldArray= this.hLayout(rows-2,columns-2,startRowIndex+1,startColIndex+1,oldArray);
            // 去掉第一个环多余座位
            oldArray= this.uLayout(startRowIndex,startColIndex,columns,rows,oldArray,orientation)
            // 去掉第二个环多余座位
            this.rectArray= this.uLayout(startRowIndex+1,startColIndex+1,columns-2,rows-2,oldArray,orientation,2)
        }else if(circleNum===3){
            // 画第一个环
            oldArray=this.hLayout(rows,columns,startRowIndex,startColIndex,oldArray);
            //画第二个环
            oldArray= this.hLayout(rows-2,columns-2,startRowIndex+1,startColIndex+1,oldArray);
            // 画第三个环
            oldArray= this.hLayout(rows-4,columns-4,startRowIndex+2,startColIndex+2,oldArray);
            // 去掉第一个环多余座位
            oldArray= this.uLayout(startRowIndex,startColIndex,columns,rows,oldArray,orientation)
            // 去掉第二个环多余座位
            oldArray= this.uLayout(startRowIndex+1,startColIndex+1,columns-2,rows-2,oldArray,orientation,2)
            // 去掉第三个环多余座位
            this.rectArray= this.uLayout(startRowIndex+2,startColIndex+2,columns-4,rows-4,oldArray,orientation,3)
        }else{
              alert('请设置环数小于等于三的正整数')
       }

        }else if( this.modelType==='gyrationShape'){
          //回型
            oldArray=this.hLayout(rows,columns,startRowIndex,startColIndex,oldArray);
            // // 去掉四个角
            //  oldArray[0][0] =this.emptyFlag
            //  oldArray[0][rows-1] =this.emptyFlag
            //   oldArray[columns-1][0] =this.emptyFlag
            //   oldArray[columns-1][rows-1] =this.emptyFlag
              this.rectArray= oldArray;

        }else if(this.modelType==='deskShape'){
           //课桌型
           this.rectArray=this.tableLayout(rows,columns,startRowIndex,startColIndex,oldArray);
        }else{

        }
     },

      /** 根据朝向 去掉环中多余的座位
       * rows:横向座位数
       * columns:纵向座位数
       * startRowIndex:左上角开始行位置
       * startColIndex:左上角开始的列位置
       * oldArray:需要标注画环的二维数组
       * orientation:朝向
       * circleNum:当前处理第几个环（从外到里依次1、2、3）
       */
      uLayout:function(startRowIndex,startColIndex,columns,rows,oldArray,orientation,circleNum=1){
         //向上
         if(orientation==='u'){
            for(let i=1;i<columns-1;i++){
                oldArray[startRowIndex][startColIndex+i] = this.emptyFlag
            }
            oldArray[startRowIndex+rows-1][startColIndex] = this.emptyFlag
            oldArray[startRowIndex+rows-1][startColIndex+columns-1] = this.emptyFlag
            if(circleNum===2){
              oldArray[startRowIndex-1][startColIndex] = this.seatType
              oldArray[startRowIndex-1][startColIndex+columns-1] = this.seatType
            }else if(circleNum===3){
              oldArray[startRowIndex-1][startColIndex] = this.seatType
              oldArray[startRowIndex-1][startColIndex+columns-1] = this.seatType
              oldArray[startRowIndex-2][startColIndex] = this.seatType
              oldArray[startRowIndex-2][startColIndex+columns-1] = this.seatType
            }
            return oldArray;
         }else if(orientation==='d'){
            //向下
            for(let i=1;i<columns-1;i++){
              oldArray[startRowIndex+rows-1][startColIndex+i] = this.emptyFlag
            }
            oldArray[startRowIndex][startColIndex] = this.emptyFlag
            oldArray[startRowIndex][startColIndex+columns-1] = this.emptyFlag

             if(circleNum===2){
              oldArray[startRowIndex+rows][startColIndex] = this.seatType
              oldArray[startRowIndex+rows][startColIndex+columns-1] = this.seatType
            }else if(circleNum===3){
              oldArray[startRowIndex+rows][startColIndex] = this.seatType
              oldArray[startRowIndex+rows][startColIndex+columns-1] = this.seatType
              oldArray[startRowIndex+rows+1][startColIndex] = this.seatType
              oldArray[startRowIndex+rows+1][startColIndex+columns-1] = this.seatType
            }
            return oldArray;
         }else if(orientation==='l'){
            //向左
            for(let i=1;i<rows-1;i++){
                oldArray[startRowIndex+i][startColIndex] = this.emptyFlag
            }
            oldArray[startRowIndex][startColIndex+columns-1] = this.emptyFlag
            oldArray[startRowIndex+rows-1][startColIndex+columns-1] = this.emptyFlag

            if(circleNum===2){
              oldArray[startRowIndex][startColIndex-1] = this.seatType
              oldArray[startRowIndex+rows-1][startColIndex-1] = this.seatType
            }else if(circleNum===3){
              oldArray[startRowIndex][startColIndex-1] = this.seatType
              oldArray[startRowIndex+rows-1][startColIndex-1] = this.seatType
              oldArray[startRowIndex][startColIndex-2] = this.seatType
              oldArray[startRowIndex+rows-1][startColIndex-2] = this.seatType
            }
            return oldArray;
         }else if(orientation==='r'){
            //向右
            for(let i=1;i<rows-1;i++){
                oldArray[startRowIndex+i][startColIndex+columns-1] = 0
            }
            oldArray[startRowIndex][startColIndex] = this.emptyFlag
            oldArray[startRowIndex+rows-1][startColIndex] = this.emptyFlag

             if(circleNum===2){
              oldArray[startRowIndex][startColIndex+columns] = this.seatType
              oldArray[startRowIndex+rows-1][startColIndex+columns] = this.seatType
            }else if(circleNum===3){
              oldArray[startRowIndex][startColIndex+columns] = this.seatType
              oldArray[startRowIndex+rows-1][startColIndex+columns] = this.seatType
              oldArray[startRowIndex][startColIndex+columns+1] = this.seatType
              oldArray[startRowIndex+rows-1][startColIndex+columns+1] = this.seatType
            }
            return oldArray;
         }else{
           console.error('选择正确的朝向');
         }

      },

      /**
       * 根据横向和纵向的座位数画环
       * rows:横向座位数
       * columns:纵向座位数
       * startRowIndex:左上角开始行位置
       * startColIndex:左上角开始的列位置
       * oldArray:需要标注画环的二维数组
       * setValue:设置环位置的数组坐标占位符
       */
      hLayout:function(rows,columns,startRowIndex,startColIndex,oldArray,setValue=1){
           for(let i=0;i<rows-1;i++){
                oldArray[startRowIndex+i][startColIndex] = setValue
                oldArray[startRowIndex+i][startColIndex+columns-1] = setValue
            }
          for(let j=0;j<columns;j++){
               oldArray[startRowIndex+rows-1][startColIndex+j] = setValue
               oldArray[startRowIndex][startColIndex+j] = setValue
          }
           return oldArray;
      },
        /**
       * 根据横向和纵向的座位数布局课桌型
       * rows:横向座位数
       * columns:纵向座位数
       * startRowIndex:左上角开始行位置
       * startColIndex:左上角开始的列位置
       * oldArray:需要标注画环的二维数组
       * setValue:设置环位置的数组坐标占位符
       */
      tableLayout:function(rows,columns,startRowIndex,startColIndex,oldArray,setValue=1){
        for(let i=0;i<rows;i++){
          for(let j=0;j<columns;j++){
                    oldArray[startRowIndex+i][startColIndex+j] = setValue
           }
        }
       return oldArray;
      },

      initRectArray: function(){
        //初始化布局
        if(this.seatModelArrayHeight){
          let rectArray = Array(this.seatModelArrayHeight).fill(this.emptyFlag).map(()=>Array(this.seatModelArrayWidth).fill(this.emptyFlag));
          this.rectArray = rectArray;
        }
      },


  },
  mounted(){
    this.initRectArray();
    this.renderSeat();
    this.$emit('initSeatModelArray', this.rectArray)
  }

}
</script>

<style scoped>

  /*画布中小正方形 */
  .square-block{
    float:left;
    display: flex;
    justify-content: center;
    align-items: center;
    border: white 5px solid;
    background: #EBEBEB;
  }

  /*画布区域大小*/
  .inner-wrapper{
    position: absolute;
    box-sizing: content-box;
    padding: 0px 10px     /* overflow :hidden; */
  }
  .drag{
      position: absolute;
      top: 0;
      left: 0;
      pointer-events: none;
      /* background-size: cover; */
      opacity: .5;
}
  .inner-seat{
    width:100%;
    height:100%;
    cursor: pointer;
  }
 .selected-seat{
    background: url('../../assets/selected.png') center center no-repeat;
    background-size: 100% 100%;
        line-height: 42px;
  }
  .unselected-seat{
    background-size: 100% 100%;
  }
</style>
