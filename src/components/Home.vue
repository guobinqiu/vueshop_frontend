<template>
  <el-container>

    <el-header>
      <div>
        <img src="../assets/logo.png" alt="">
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>

    <el-container>
      <el-aside width="200px">
        <el-menu
        background-color="#333744"
        text-color="#fff"
        active-text-color="#409eff"
        unique-opened
        router
        :default-active="activePath">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id.toString()" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i class="el-icon-menu"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="subitem.path" v-for="subitem in item.children" :key="subitem.id"
            @click="saveNavState(subitem.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{subitem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>

      <el-main>
        <router-view></router-view>
      </el-main>

    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      menuList: [],
      activePath: ''
    }
  },
  // 初始化推荐在created,此时dom还没创建, mounted的话dom已经创建就有点晚
  created() {
    this.getMenuList()
  },
  mounted() {
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    async getMenuList() {
      try {
        const { data: res } = await this.$http.get('menus')
        // console.log(res)
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.menuList = res.data
      } catch (err) {
        this.$message.error(err)
      }
    },
    // 保存菜单链接的激活状态
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
    }
  }
}
</script>

<style lang="less" scoped>
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
  color: #fff;
  font-size: 20px;
  div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
  img {
    width: 50px;
    height: 50px;
  }
}
.el-aside{
  background-color: #333744;
  .el-menu {
    border-right: none;//去菜单边框竖条
  }
}
.el-main{
  background-color: #eaedf1;
}
.el-container{
  height: 100%;
}
</style>
