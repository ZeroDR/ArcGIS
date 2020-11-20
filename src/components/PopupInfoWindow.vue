<template>
  <!-- <MountingPortal mountTo="#popupinfowindow"> -->
    <div>
      {{type}}
    <!-- <component :is="type" :target="target" :sourceInfo="sourceInfo"></component> -->
     </div>
  <!-- </MountingPortal> -->
</template>
<script>
  export default {
    name: 'PopupInfoWindow',
    props:['type','target','sourceInfo'],
    components:{
    },
    data () {
      return {
      }
    },
    watch:{
      type(val){
        console.log(val);
      }
    },
    beforeCreate(){
    },
    created(){
    },
    mounted(){
      console.log('hhhh'+123);
    },
    methods: {
      
    }
  };
</script>
<style lang="scss" scoped>
</style>
