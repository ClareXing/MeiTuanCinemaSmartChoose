<template>
  <div  class="drag" :style="{width:facilityWidth*gridSize+'px',
                            height:facilityHeight*gridSize+'px',
     top:gridSize*topIndex+'px',left:gridSize*leftIndex+'px'}">
     <img :src="picPath" width="100%" height="100%">
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
      picPath:'',
  }
},
methods: {
        test:function(){
           this.picPath= require("../../assets/" +this.facilityType)
        }
   },

  mounted(){
    this.test()
    // this.$emit('initSeatModelArray', this.rectArray)
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
