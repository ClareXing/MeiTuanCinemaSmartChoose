<template>
  <div>
  <div class='divWall' :style=""></div>
  <slot name='door'></slot>
  </div>
</template>

<script>
export default {

  props:['','','',''],

  data(){
    return{
       
    }
  },



}
</script>

<style scoped>
  .divWall{
    position: relative;
  }

</style>>

