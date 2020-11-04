<template>
  <div class="inner-seat" :class="seatStyle">
      {{label}}
  </div>
</template>

<script>
export default {
 props: {
    seatType:String ,//座位类型（普通座位，主席座等）
    seatNo:String , //座位编号
    seatLable:String ,//座位文字显示信息（与座位号二者只能显示其中一个）
    seatOrientation:String //座位朝向（上下左右 依次 u d l r）默认朝上
 },
 data(){
   return{
      label:'',//显示文字信息
      showLable:false,//显示文字信息
      showNo:true//显示编号
   }
 },
 methods:{
     setLabel:function(){
       if(this.showLable){
         this.label=this.seatLable;
       }else if(this.showNo){
         this.label= this.seatNo
       }else{
         this.label=''
       }
     }
 },
computed:{
   seatStyle: function () {
    return {
      'selected-seat': this.seatType==='1' && this.seatOrientation === 'u',
      'unselected-seat':  this.seatType==='2' && this.seatOrientation === 'd'
    }
}
}
}
</script>

<style scoped>
.inner-seat{
    width:100%;
    height:100%;
    cursor: pointer;
  }
 .selected-seat{
    background: url('../../assets/selected.png') center center no-repeat;
    background-size: 100% 100%;
  }
  .unselected-seat{
    background: red;
    background-size: 100% 100%;
  }

</style>
