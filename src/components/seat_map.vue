<template>
  <div class="wrapper">
    <div class="seat-wrapper">
      <div class="inner-seat-wrapper" ref="innerSeatWrapper">
        <div v-for="seat in seats" :key="seat"   >
          <!--这里的v-if很重要，如果没有则会导致报错，因为seatArray初始状态为空-->
          <div v-for="col in seatCol" :key="col" v-if="seatArray.length>0"
            class="square-block" :style="{width:seatSize+'px',height:seatSize+'px'}">
              <seat :v-if="seatArray[row-1][col-1]!==0" :seatType='seatArray[row-1][col-1].toString()' seatOrientation='u'/>
          </div>
        </div>
        <!--设施-->
        <facility/>
      </div>
    </div>
  </div>
</template>
<script>
import seat from './customComponents/seat.vue'
import facility from './customComponents/facility.vue'
export default {
    name: 'seatMap',
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
    props:{
       unitSize:Number, //单元格的大小
       // 座位设施以及画布长度宽度
       blockWidth: Number ,//围墙长
       blockHeight: Number,//围墙高度:
       facilitys:Array,//设备信息 facilityInfo
       seats:Array,// 座位信息
    },
		data () {
			return {

      }

		},
    methods:{

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
