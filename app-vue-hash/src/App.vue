<template>
  <div id="app">
    <keep-alive :include="loadedRouteNames">
      <router-view/>
    </keep-alive>
  </div>
</template>

<script>
export default {
  data() {
    return {
      loadedRouteNames: []
    }
  },
  mounted() {
    if (window.__POWERED_BY_QIANKUN__) {
      this.parentProps.onGlobalStateChange(state => {
        const { childRoute } = state['app-vue-hash'];
        // hash 模式需要去掉路由前面的 #/
        console.log('childRoute', childRoute);
        const loadedRoutes = childRoute.map(item => this.$router.resolve(item.slice(2)));
        console.log('loadedRoutes', loadedRoutes);
        const loadedRouteNames = loadedRoutes.map(item => item.route.name);
        console.log('loadedRouteNames', loadedRouteNames);
        this.loadedRouteNames = loadedRouteNames;
      }, true);
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #42b983;
}
span{
  font-weight: bold;
  color: #2c3e50;
  cursor: pointer;
  margin: 0 15px;
  text-decoration: underline;
}
</style>
