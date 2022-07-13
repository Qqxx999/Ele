<!--  -->
<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="14">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" 
          clearable
          @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList">
            </el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
    </el-card>
    <!-- 添加用户的对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible" width="50%"
      @close="addDialogClosed">
      <!-- 内容主体区域 -->
      <el-form ref="form" :model="addForm" :rules="addFormRules" label-width="70px">
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
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
      <el-button @click="addDialogVisible = false">取 消</el-button>
      <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%" @close="editDialogClosed">
      <el-form ref="form" :model="editForm" :rules="editFormRules" label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile"></el-input>
        </el-form-item>
      </el-form>  
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配角色对话框 -->
    <el-dialog
      title="分配角色对话框"
      :visible.sync="setRoleDialogVisable"
      width="50%" @close="setRoleDialogClosed">
      <div>
        <p>当前用户: {{userinfo.username}}</p>
        <p>当前角色：{{userinfo.role_name}}</p>
        <p>分配新角色
          <el-select v-model="selectedRoleId" placeholder="请选择">
            <el-option
              v-for="item in rolelist"
              :key="item.id"
              :label="item.roleName"
              :value="item.id">
            </el-option>
          </el-select>
        </p>
      </div>
     <span slot="footer" class="dialog-footer">
       <el-button @click="setRoleDialogVisable = false">取 消</el-button>
       <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
     </span>
    </el-dialog>
    <!-- 用户列表区 -->
    <el-table :data="userList" border stripe>
      <el-table-column type="index">
      </el-table-column>
      <el-table-column label="姓名" prop="username">
      </el-table-column>
       <el-table-column label="邮箱" prop="email">
      </el-table-column>
       <el-table-column label="电话" prop="mobile">
      </el-table-column>
       <el-table-column label="角色" prop="role_name">
      </el-table-column>
       <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)">
            </el-switch>
          </template>  
      </el-table-column>
       <el-table-column label="操作" width="180px">
         <template slot-scope="scope">
           <!-- 修改按钮 -->
           <el-button type="primary" icon="el-icon-edit" 
           size="mini" @click="showEditDialog(scope.row.id)"></el-button>
           <!-- 删除按钮 -->
           <el-button type="danger" @click="removeUserById(scope.row.id)" icon="el-icon-delete" size="mini">

           </el-button>
           <!-- 分配角色按钮 -->
           <el-tooltip class="item" content="分配角色" placement="top" :enterable="false">
             <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
           </el-tooltip>
         </template>
      </el-table-column>
    </el-table>
    <!-- 分页区域 -->
     <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[1, 2, 5, 10]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </div>
</template>

<script>
export default {
  data () {
    // 验证邮箱的规则
    var checkEmail = (rule, value, cb) => {
      const reEmail = /^([A-Za-z0-9_\-.])+@([A-Za-z0-9_\-.])+\.([A-Za-z]{2,4})$/
      if (reEmail.test(value)) {
        // 合法邮箱
        return cb()
      }
      cb(new Error('请输入正确的邮箱'))
    }
    // 验证手机号的规则
    var checkMobile = (rule, value, cb) => {
      const regMobile = /^(13[0-9]|14[5|7]|15[0|1|2|3|4|5|6|7|8|9]|18[0|1|2|3|5|6|7|8|9])\d{8}$/
      if (regMobile.test(value)) {
        return cb()
      }
      cb(new Error('请输入正确的手机号'))
    }
    return {
      // 获取用户列表的参数对象
      queryInfo: {
        query: '',
        // 当前的页数
        pagenum: 1,
        // 当前每页显示的数据
        pagesize: 2
      },
      userList: [],
      total: 0,
      // 控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表单的验证规则对象
      addFormRules: {
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: 'blur'
          }, 
          {
            min: 3,
            max: 10,
            message: '用户名长度在3-10个字符之间',
            trigger: 'blur'
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          }, 
          {
            min: 3,
            max: 10,
            message: '用户名长度在6-15个字符之间',
            trigger: 'blur'
          }
        ],
        email: [
          {
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          }, 
          {
            validator: checkEmail,
            trigger: 'blur'
          }
        ],
        mobile: [
          {
            required: true,
            message: '请输入手机号',
            trigger: 'blur'
          }, 
          {
            validator: checkMobile,
            trigger: 'blur'
          }
        ]
      },
      // 控制修改对话框的显示与隐藏
      editDialogVisible: false,
      // 查询到的用户信息对象
      editForm: {},
      // 修改表单的验证规则对象
      editFormRules: {
         email: [
          {
            required: true,
            message: '请输入邮箱',
            trigger: 'blur'
          }, 
          {
            validator: checkEmail,
            trigger: 'blur'
          }
        ],
        mobile: [
          {
            required: true,
            message: '请输入手机号',
            trigger: 'blur'
          }, 
          {
            validator: checkMobile,
            trigger: 'blur'
          }
        ]
      },
      // 控制分配角色对话框的显示与隐藏
      setRoleDialogVisable: false,
      // 需要被分配角色的用户信息
      userinfo: {},
      // 所有角色的数据列表
      rolelist: [],
      // 已选中的角色id值
      selectedRoleId: ''
    }
  },
  created() {
    this.getUserList()
  },
  components: {},

  computed: {},

  methods: {
    async getUserList() {
    const { data: res} = await this.$axios.get('users',{ params: this.queryInfo })
    if(res.meta.status !== 200) return this.$message.error('获取数据失败');
    this.userList = res.data.users
    this.total = res.data.total
    },
    // 监听pagesize改变的事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听页码值改变的事件
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听switch开关状态的改变
    async userStateChange(userinfo) {
      console.log(userinfo);
    const {data: res} = await this.$axios.put(
      `users/${userinfo.id}/state/${userinfo.mg_state}`)
    if(res.meta.status !== 200) {
    userinfo.mg_state = !userinfo.mg_state
    return this.$message.error('更新用户状态失败')
    }
    this.$message.success('更新用户状态成功')
   },
   // 监听添加用户对话框的关闭事件
   addDialogClosed() {
     this.$refs.form.resetFields()
   },
   // 点击按钮，添加新用户
   addUser() {
     this.$refs.form.validate(async valid => {
       if(!valid) return 
       // 可以发起添加用户的网络请求
      const {data: res} = await this.$axios.post('users', this.addForm)
      if(res.meta.status !== 201) {
        this.$message.error('添加用户失败')
      }
      this.$message.success('添加用户成功')
      // 隐藏添加用户的对话框
      this.addDialogVisible = false
      // 重新获取用户的列表数据
      this.getUserList()
     })
   },
   // 展示编辑用户的对话框
   async showEditDialog(id) {
    const{data: res} = await this.$axios.get('users/' + id)
    if (res.meta.status !== 200) {
      return this.$message.error('查询用户信息失败')
    }
     this.editForm = res.data
     this.editDialogVisible = true
    },
    // 监听用户修改对话框的关闭事件
    editDialogClosed() {
      this.$refs.form.resetFields()
    },
    // 修改用户信息并提交
    editUserInfo() {
      this.$refs.form.validate(async valid => {
        if(!valid) return
        // 发起修改用户信息的数据请求
        const {data: res} = await this.$axios.put('users/' + this.editForm.id, {
          email: this.editForm.email,
          mobile: this.editForm.mobile
        })
        if (res.meta.status !== 200) {
          return this.$message.error('更新用户信息失败')
        }
        // 关闭对话框
        this.editDialogVisible = false
        // 刷新数据列表
        this.getUserList()
        // 提示修改成功
        this.$message.success('更新用户信息成功')
      })
    },
    // 根据id删除对应的用户信息
    async removeUserById(id) {
      // 弹框询问是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).catch(err => {
        return err
      })
      if(confirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const {data: res} = await this.$axios.delete('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功')
      this.getUserList()
    },
    //展示分配角色的对话框
    async setRole(userinfo) {
      this.userinfo = userinfo
      // 展示对话框之前获取所有角色列表
      const {data: res} = await this.$axios.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }

      this.rolelist = res.data

      this.setRoleDialogVisable = true
    },
    // 点击按钮分配角色
    async saveRoleInfo() {
      if (!this.selectedRoleId) {
        return this.$message.error('请选择分配的角色')
      }
    const {data: res} = await this.$axios.put(`users/${this.userinfo.id}/role`, {rid: this.selectedRoleId})
    if (res.meta.status !== 200) {
      return this.$message.error('更新角色失败') 
     }
     this.$message.success('更新角色成功')
     this.getUserList()
     this.setRoleDialogVisable = false
    },
    // 分配角色对话框的关闭事件
    setRoleDialogClosed() {
      this.selectedRoleId = '',
      this.userinfo = ''
    }
  }
}

</script>
<style lang="less" scoped>
</style>