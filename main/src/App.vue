<template>
  <el-container>
    <el-aside width="200px">
      <el-menu @select="changeMenu" :default-active="currentTab" router>
        <el-menu-item v-for="item in menuData" :index="item.value"  :key="item.value">
          <template #title>{{ item.name }}</template>
        </el-menu-item>
      </el-menu>
    </el-aside>
    <el-main>
      <el-tabs v-model="currentTab" type="card" closable @tab-remove="removeTab" @tab-click="changeTab">
        <el-tab-pane
          v-for="item in allTabs"
          :key="item.value"
          :label="item.name"
          :name="item.value"
        >
        </el-tab-pane>
      </el-tabs>
      <!-- 循环生成容器，每一个微应用使用独立的容器，自己控制微应用的显隐 -->
      <div 
        v-for="item in microApps" 
        :key="item.name" 
        :id="item.container.slice(1)" 
        v-show="$route.path.startsWith(item.prefixPath)"
      ></div>
      <!-- 缓存主应用的路由 -->
      <keep-alive>
        <router-view></router-view>
      </keep-alive>
    </el-main>
  </el-container>
</template>

<script>
import { loadMicroApp, initGlobalState } from 'qiankun';
// 初始化 state
const actions = initGlobalState({});
export default {
  data() {
    return {
      menuData: [
        { name: '主应用的home', value: '/home' },
        { name: '主应用的about', value: '/about' },
        { name: '子应用1的home', value: '/app-vue-hash/#/' },
        { name: '子应用1的about', value: '/app-vue-hash/#/about' },
        { name: '子应用2的home', value: '/app-vue-history/' },
        { name: '子应用2的about', value: '/app-vue-history/about' },
      ],
      allTabs: [],
      currentTab: '',
      loadedApp: {}, // 已加载的微应用
      microApps: [
        { 
          name: 'app-vue-hash', 
          entry: 'http://localhost:1111', 
          container: '#appContainer1',
          prefixPath: '/app-vue-hash',
        },
        { 
          name: 'app-vue-history',
          entry: 'http://localhost:2222', 
          container: '#appContainer2',
          prefixPath: '/app-vue-history',
        }
      ], // 微应用的数据
    }
  },
  methods: {
    changeMenu (indexPath) {
      console.log('indexPath', indexPath)
      if (this.currentTab === indexPath) return;
      const existTab = this.allTabs.find(item => item.value === indexPath);
      // tabList已经存在了
      if (existTab) {
        this.currentTab = existTab.value;
      } else {
        // 先判断是子应用还是主应用，再判断子应用是否已加载
        const microApp = this.microApps.find(item => indexPath.includes(item.prefixPath));
        // 如果是子应用
        if (microApp) {
          // '/app-vue-history/about'.replace('/app-vue-history', '')
          // '/about'
          const childRoutePath = indexPath.replace(microApp.prefixPath,'');
          // 如果目前没有加载该子应用
          // 我没有加这个判断，所以从主应用跳到app1-home，在从app1-home跳到app1-about就会重新加载app1
          if (!this.loadedApp[microApp.name]) {
            console.log('加载子应用', microApp)
            const app = loadMicroApp(microApp); // 多了个参数 prefixPath，但是可以忽略
            this.loadedApp[microApp.name] = {
              app
            };
          }
          // 如果子应用已加载，将子应用的路由纪录到数组中，并通知子应用增加 keep-alive 的 include
          this.loadedApp[microApp.name].childRoute.push(childRoutePath);
          actions.setGlobalState(this.loadedApp);
        }
        // 添加 tab 纪录
        const selectMenu = this.menuData.find(item => item.value === indexPath);
        if(selectMenu){
          this.allTabs.push(selectMenu);
          this.currentTab = indexPath;
        }
      }
    },
    removeTab (tab) {
      // 先判断是子应用还是主应用
      const microApp = this.microApps.find(item => tab.includes(item.prefixPath));
      if (microApp) {
        // 移除子应用已缓存的应用
        const childRoutePath = tab.replace(microApp.prefixPath,'');
        const childRouteIndex = this.loadedApp[microApp.name].childRoute.indexOf(childRoutePath);
        this.loadedApp[microApp.name].childRoute.splice(childRouteIndex, 1);
        actions.setGlobalState(this.loadedApp);
        // 再当前微应用的页面是否已全部关闭
        if (this.loadedApp[microApp.name].childRoute.length === 0) {
          this.loadedApp[microApp.name].app.unmount();
          this.loadedApp[microApp.name] = null;
        }
      }
      const deleteTab = this.allTabs.findIndex(item => item.value === tab);
      // 如果被关闭的 tab 是激活的 tab
      if (tab === this.currentTab) {
        let defaultPath = '/';
        if (this.allTabs[deleteTab+1]) {
          defaultPath = this.allTabs[deleteTab+1].value
        } else if (this.allTabs[deleteTab-1]) {
          defaultPath = this.allTabs[deleteTab-1].value
        }
        this.currentTab = defaultPath;
        this.changeTab({ name: defaultPath });
      }
      this.allTabs.splice(deleteTab, 1);
    },
    changeTab (tab) {
      if(tab.name === this.$route.path) return;
      this.$router.push(tab.name);
    },
    initTab(){
      let { fullPath } = this.$route;
      const { pathname } = window.location;
      // 主应用的路由页面
      if (fullPath === '/' && pathname !==  '/') {
        fullPath = pathname;
      }
      // history 子应用路由页面，需要去掉 #/
      if (fullPath.includes('app-vue-history') && fullPath.includes('#')) {
        fullPath = pathname
      }
      this.changeMenu(fullPath);
    }
  },
  mounted() {
    this.initTab();
  },
}
</script>

<style scoped>
#app{
  height: 100vh;
  text-align: center;
  position: relative;
}
header>a{
  margin: 0 20px;
}
.appContainer{
  background: #ccc;
  padding: 20px;
}
</style>
