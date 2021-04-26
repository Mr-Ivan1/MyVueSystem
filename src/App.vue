<template>
  <div id="app">
    <div id="nav">
      <router-link to="/"></router-link>
    </div>
    <router-view/>
  </div>
</template>

<style lang="less">
html,body,#app{
  height: 100%;
  min-width: 1366px;
}
*{
  margin: 0;
  padding: 0;
}
.el-breadcrumb{
  margin-bottom: 20px;
}
.ql-editor{
  min-height: 350px ;
}
</style>
