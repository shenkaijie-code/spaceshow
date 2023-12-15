<template>
  <el-container>
    <!--    侧-->
<!--    <el-aside style="width:180px;"></el-aside>-->
    <el-container>
      <!--      顶-->
      <el-header>空间展示系统</el-header>
      <!--  内容区域-->
      <el-main>
        <Home/>
      </el-main>
      <!--      底部-->
      <el-footer></el-footer>
    </el-container>
  </el-container>
</template>
<script>

import Home from "@/views/home/Home.vue";

export default {
  name: 'Layout',
  components: {Home},
  // 获取分类数据
  setup() {
    // const store = useStore()
    // store.dispatch('category/getList')
  }
}
</script>
<style scoped>
html, body, #app {
  height: 100%;
  margin: 0;
  padding: 0;
}

.el-container {
  height: 100%;
}

.el-header,
.el-footer {
  background-color: #b3c0d1;
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-aside {
  background-color: #d3dce6;
  color: #333;
  text-align: center;
  line-height: 100px;
}

.el-main {
  background-color: #e9eef3;
  color: #333;
  text-align: center;
  line-height: 20px;
}

.el-menu {
  background-color: #d3dce6;
}
</style>
