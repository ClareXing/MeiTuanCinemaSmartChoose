<template>
  <div>
    <div class='divWall' :v-show="isShow" :style="{top:startTopIndex*borderWidth+'px',
                                                  left:startLeftIndex*borderWidth+'px',
                                                  width:wallWidth+'px',
                                                  height:wallHeight+'px',
                                                  border:borderWidth+'px'+ ' red solid'}"></div>
    <slot name='door'></slot>
  </div>
</template>

<script>
export default {

  props:['isShow','startTopIndex','startLeftIndex','wallWidth','wallHeight','borderWidth'],

  data(){
    return{

    }
  },

  methods: {

  }

}
</script>

<style scoped>
  .divWall{
    position: relative;
  }

</style>>

