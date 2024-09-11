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
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEdit(data.row.id)"></el-button>
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
      width="50%"
      @close="addDialogClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username" >
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    // 自定义email验证器
    const checkEmail = (rule, value, cb) => {
      const regexp = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regexp.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的邮箱'))
    }

    // 自定义mobile验证器
    const checkMobile = (rule, value, cb) => {
      const regexp = /^(13[0-9]|14[5-9]|15[0-3,5-9]|16[6]|17[0-8]|18[0-9]|19[8-9])\d{8}$/
      if (regexp.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的手机号'))
    }

    return {
      userList: [],
      total: 0,
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 2
      },
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '长度在 3 到 10 个字符',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            min: 6,
            max: 15,
            message: '长度在 6 到 15 个字符',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      editForm: {},
      editFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          {
            min: 3,
            max: 10,
            message: '长度在 3 到 10 个字符',
            trigger: 'blur'
          }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      }
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
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        try {
          const { data: res } = await this.$http.post('users', this.addForm)
          if (res.meta.status !== 201) return this.$message.error(res.meta.msg)
          this.$message.success(res.meta.msg) // 提示成功
          this.addDialogVisible = false // 关闭对话框
          this.getUserList() // 刷新数据列表
        } catch (err) {
          this.$message.error(err)
        }
      })
    },
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    async showEdit(id) {
      // console.log('edit', id)
      const { data: res } = await this.$http.get('/users/' + id)
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      this.editForm = res.data
      this.editDialogVisible = true
    },
    editUser() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        try {
          const { data: res } = await this.$http.put('users/' + this.editForm.id, {
            email: this.editForm.email,
            mobile: this.editForm.mobile
          })
          if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
          this.$message.success(res.meta.msg)
          this.editDialogVisible = false
          this.getUserList()
        } catch (err) {
          this.$message.error(err)
        }
      })
    },
    del(id) {
      // console.log('del', id)
      this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete('users/' + id)
        if (res.meta.status !== 200) {
          return this.$message({
            type: 'error',
            message: res.meta.msg
          })
        }
        this.$message({
          type: 'success',
          message: '删除成功!'
        })
        this.getUserList()
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    }
  }
}
</script>

<style lang="less" scoped>
</style>
