<template>
  <div id="app">
    <MainView :global="global" v-if="global"></MainView>
  </div>
</template>

<script>
import MainView from './components/MainView.vue'

export default {
  name: 'app',
  components: {
    MainView
  },
  data(){
    return {
      global:null,
      auth: null
    }
  },
  mounted(){
    setTimeout(()=>{
      this.global = this.m3.global;
      this.auth = this.m3.auth.signedUser;
      this.m3.setTitle(this.auth);
    },500)
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  font-size: 12px;
}

.el-checkbox__label {
    font-size: 12px!important;
}
</style>
