<template>
  <div v-drag class="drag" >
        <div class="dragImg" ref="dragImg" :v-show="enableDrag">
                <img src="../../assets/Rostrum.png" width="100%" height="100%">
       </div>
  </div>

</template>

<script>
export default {
  props:{
    initWidth:String,
    initHeight:String,
    startLeft:String,
    startTop:String,
  },
data () {
  return {
  }
},
directives:{
    drag(el){
        el.onmousedown = function(e){
          console.log(`e.pageX:${e.pageX },e.pageY:${e.pageY},el.offsetLeft:${el.offsetLeft},el.offsetTop:${el.offsetTop}`)
            var disx = e.pageX - el.offsetLeft;//offsetLeft 初始位置左上偏移量
            var disy = e.pageY - el.offsetTop;
            document.onmousemove = function (e){
                el.style.left = e.pageX - disx+'px';
                el.style.top = e.pageY - disy+'px';
            }
            document.onmouseup = function(){
                document.onmousemove = document.onmouseup = null;
            }
        }
  }
},
methods: {

   }
}
</script>

<style>
.drag{
      width: 100px;
      height: 100px;
      position: absolute;
      top: 0;
      left: 0;
      background:url(../../assets/zhuxitai.png) no-repeat;
      background-size: cover;
      opacity: .6;
}
</style>
