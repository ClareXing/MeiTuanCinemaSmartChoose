<template>
  <div  class="drag" :style="{width:facilityWidth*gridSize+'px',
                            height:facilityHeight*gridSize+'px',
     top:gridSize*topIndex+'px',left:gridSize*leftIndex+'px'}">
     <img :src="imgSrc"  width="100%" height= "100%" >
  </div>
</template>

<script>
export default {
  props:{
    gridSize:Number,//单元格大小
    facilityType:String,//设备类型
    facilityWidth:Number,
    facilityHeight:Number,
    holdSeatNum:{
      type:Number,
      default:0
    },
    topIndex:{
      type:Number,
      default:2
    },
    leftIndex:{
      type:Number,
      default:2
    },
    facilityOrientation:String,//朝向
  },
data () {
  return {
      imgSrc:'',
  }
},
methods: {
        // 初始化图片路径
        initImgSrc:function(){
           this.imgSrc= require("../../assets/" +this.facilityType)
           console.log(this.facilityWidth,this.facilityHeight,this.gridSize)
        },
   },

  mounted(){
    this.initImgSrc();

  }
}
</script>

<style>
.drag {
    position: absolute;
    pointer-events: none;
    background: #fff;
    opacity: .6;
}

</style>
