<template>
  <el-container class="container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/heima.png"
             alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button @click="logOut">退出</el-button>
    </el-header>
    <!-- 内容主题区域 -->
    <el-container>
      <!-- 左侧边栏 -->
      <el-aside :width="isCollapse? '64px':'200px'">
        <!-- 侧边栏折叠按钮 -->
        <div class="toggle"
             @click="toggleAside">|||</div>
        <!-- 侧边栏内容区域 -->
        <el-menu background-color="#333744"
                 text-color="#fff"
                 active-text-color="#409EFF"
                 unique-opened
                 :collapse="isCollapse"
                 :collapse-transition="false"
                 :router="true"
                 :default-active="defaultActive">
          <!-- 一级标题 -->
          <el-submenu :index="item.id+''"
                      v-for="item in menuList"
                      :key="item.id">
            <template slot="title">
              <i :class="iconObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级标题 -->
            <el-menu-item :index="'/'+value.path"
                          v-for="value in item.children"
                          :key="value.id"
                          @click="changeDefault('/'+value.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{value.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容区域 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      menuList: [],
      iconObj: {
        125: 'iconfont icon-users',
        103: 'iconfont icon-tijikongjian',
        101: 'iconfont icon-shangpin',
        102: 'iconfont icon-danju',
        145: 'iconfont icon-baobiao'
      },
      isCollapse: false,
      defaultActive: ''
    }
  },
  created: function () {
    this.creatMenu()
    this.defaultActive = window.sessionStorage.getItem('defaultActive')
  },
  methods: {
    // 退出登录按钮
    logOut: function () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 页面生成时，获取侧边栏内容
    creatMenu: async function () {
      const { data: res } = await this.$http.get('menus')
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },
    // 切换侧边栏折叠状态
    toggleAside: function () {
      this.isCollapse = !this.isCollapse
    },
    // 切换侧边栏激活的二级标题
    changeDefault: function (path) {
      this.defaultActive = path
      window.sessionStorage.setItem('defaultActive', path)
    }
  }
}
</script>

<style lang="less" scoped>
.container {
  height: 100%;
  .el-header {
    background: #373d41;
    div {
      float: left;
      img {
        float: left;
      }
      span {
        float: left;
        margin-left: 15px;
        line-height: 60px;
        text-align: center;
        color: #fff;
        font-size: 20px;
      }
    }
    .el-button {
      margin-top: 10px;
      float: right;
    }
  }
  .el-aside {
    background: #333744;
    .toggle {
      height: 24px;
      background: #4a5064;
      color: #fff;
      line-height: 24px;
      text-align: center;
      letter-spacing: 0.2em;
      cursor: pointer;
    }
    .el-menu {
      border: none;
    }
  }
  .el-main {
    background: #eaedf1;
  }
  .el-submenu i {
    margin-right: 10px;
  }
}
</style>
