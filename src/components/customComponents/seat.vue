<template>
  <div div class="inner-seat" :class="seatStyle">
    <span style="inner-info" v-if='seatType==="1"'> {{seatLable}}</span>
  </div>
</template>

<script>
export default {
 props:['seatType','seatNo','seatLable','seatOrientation'],

 data(){
   return{

   }
 },
 methods:{

 },

computed:{
   seatStyle: function () {
    return {
      'selected-seat': this.seatType==='1' && this.seatOrientation === 'u',
      'unselected-seat':  this.seatType==='2' && this.seatOrientation === 'd'
    }
   },
  // seatInfo: function () {
  //   if(this.)
  //     return this.firstName + ' ' + this.lastName
  //   }
  }
}
</script>

<style  scoped>
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
    background: red;
    background-size: 100% 100%;
  }
  .inner-info{
    line-height: 15px; width: 70%;
  }

</style>
