<template>
  <div v-drag class="drag" >
  </div>
</template>

<script>
export default {
    props:{
    initWidth:String,
    initHeight:String,
  },
data () {

return {
}
},
directives:{
    drag(el,bindings){
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
      background-size: cover
}
</style>
