<template>
  <div id="app">
    <mains></mains>
  </div>
</template>

<script>
import mains from '@/components/main.vue'
export default {
  name: "App",
  components:{
    mains
  }
  
  
};
</script>

<style>


html,
body,
#app {
  height: 100%;
}

#title {
  position: absolute;
  top: 0;
  color: white;
  font-family: "HYShangWeiShouShuW";
  font-weight: normal;
  font-size: 30px;
}

@media screen and (max-width: 500px) {
  #title {
    font-size: 20px;
  }
}

@media screen and (min-width: 1000px) {
  #title {
    font-size: 45px;
  }
}
</style>
