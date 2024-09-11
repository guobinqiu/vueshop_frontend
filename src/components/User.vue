<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片 -->
    <el-card>
      <!-- 栅格布局 -->
      <el-row :gutter="20">
        <!-- 第一列 -->
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>

        <!-- 第二列 -->
        <el-col :span="4" class="aaa">
          <el-button type="primiary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>

      <!-- 展示用户列表 -->
      <!-- <ul>
        <li v-for="user in userList" :key="user.id">
          {{ user.username }}
        </li>
      </ul>
      Total: {{ total }} -->

      <el-table :data="userList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>

        <!-- 子组件(el-table-column)内部通过<slot :row="行数据"></slot>提供父组件(User.vue)数据 -->
        <el-table-column label="状态">
          <template slot-scope="data">
            <!-- {{ data.row }} -->
            <el-switch v-model="data.row.mg_state" @change="userStateChange(data.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="data">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="edit(data.row.id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="del(data.row.id)"></el-button>
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini" @click="del(data.row.id)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 3, 4]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>
    </el-card>

    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%">

      <el-form label-width="100px">
        <el-form-item label="活动名称">
          <el-input></el-input>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>

  </div>

</template>
<script>
export default {
  data() {
    return {
      userList: [],
      total: 0,
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      addDialogVisible: false
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    getUserList() {
      // 不用async/await了,换种写法,用promise写法
      this.$http.get('/users', { params: this.queryInfo }).then(resp => {
        // console.log(resp)
        const res = resp.data
        if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
        this.userList = res.data.users
        this.total = res.data.total
      }).catch(err => {
        this.$message.error(err)
      })
    },
    handleSizeChange(val) {
      this.queryInfo.pagesize = val
      this.getUserList()
    },
    handleCurrentChange(val) {
      this.queryInfo.pagenum = val
      this.getUserList()
    },
    async userStateChange(userinfo) {
      // console.log(userinfo)
      try {
        const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
        if (res.meta.status !== 200) {
          userinfo.mg_state = !userinfo.mg_state
          return this.$message.error('更新用户状态失败')
        }
        this.$message.success('更新用户状态成功')
      } catch (err) {
        this.$message.error(err)
      }
    },
    edit(id) {
      console.log('edit', id)
    },
    del(id) {
      console.log('del', id)
    }
  }
}
</script>

<style lang="less" scoped>
</style>
