<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
        <!-- 添加角色按钮 -->
        <el-row>
            <el-col>
                <el-button type="primary" @click="isDialogVisible = true">添加角色</el-button>
            </el-col>
        </el-row>
        <!-- 角色列表区域 -->
        <el-table :data="rolesList" border stripe>
            <el-table-column type="expand">
                <template slot-scope="scope">
                    <el-row :class="['bdbottom','vcenter',i1===0 ? 'bdtop' :'']" v-for="(item1 , i1) in scope.row.children" :key="item1.id">
                        <!-- 渲染一级权限 -->
                        <el-col :span="5">
                        <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                        <i class="el-icon-caret-right"></i>
                        </el-col>
                        <!-- 渲染二级和三级权限 -->
                        <el-col :span="19">
                            <!-- 渲染二级权限 -->
                            <el-row v-for="(item2 , i2) in item1.children" :key="item2.id" :class="[i2===0 ? '' :'bdtop','vcenter']">
                              <el-col :span="6">
                                  <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                                  <i class="el-icon-caret-right"></i>
                              </el-col>
                              <!-- 渲染三级权限 -->
                              <el-col :span="18">
                                  <el-tag type="warning" closable v-for="item3 in item2.children" :key="item3.id" @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>
                              </el-col>
                            </el-row>
                        </el-col>
                    </el-row>
                </template>
            </el-table-column>
            <el-table-column type="index" label="序号"></el-table-column>
            <el-table-column label="角色名称" prop="roleName"></el-table-column>
            <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
            <el-table-column label="操作" min-width="300px">
                <template slot-scope="scope">
                    <el-button size="mini" type="primary" icon="el-icon-search" @click="editRoles(scope.row.id)">编辑</el-button>
                    <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRolesById(scope.row.id)">删除</el-button>
                    <el-button size="mini" type="warning" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                </template>
            </el-table-column>

        </el-table>
    </el-card>
    <!-- 添加角色按钮视图 -->
    <el-dialog
      title="添加角色"
      :visible.sync="isDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主体 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="70px"
      >
        <el-form-item label="名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 下方区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="isDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑角色视图区 -->
     <el-dialog
      title="编辑角色"
      :visible.sync="isEditDialogVisible"
      width="50%"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="70px"
      >
        <el-form-item label="名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="isEditDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限视图 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClose"
    >
      <!-- 树形控件 -->
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
    </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: [],
      addForm: {
        roleDesc: '',
        roleName: ''
      },
      editForm: {},
      addFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ]
      },
      editFormRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ]
      },
      isDialogVisible: false,
      isEditDialogVisible: false,
      // 分配权限
      setRightDialogVisible: false,
      rightslist: [],
      // 树形控件
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中
      defKeys: [],
      roleId: ''

    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
    },
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    // 增加角色复制增加用户代码简单修改
    addUser () {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return
        // 成功后发送添加请求
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 201) {
          this.$message.error(res.meta.msg)
        } else {
          this.$message.success('添加角色成功')
        }
        this.isDialogVisible = false
        this.getRolesList()
      })
    },
    // 打开编辑框
    async editRoles (id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.editForm = res.data
      this.isEditDialogVisible = true
    },
    // 提交编辑角色
    editUserInfo () {
      this.$refs.editFormRef.validate(async (valid) => {
        if (!valid) return
        // 修改请求
        const { data: res } = await this.$http.put(
          'roles/' + this.editForm.roleId, this.editForm)
        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg)
        } else {
          // 关闭对话框
          this.isEditDialogVisible = false
          //   更新数据表
          this.getRolesList()
          //   提示成功
          this.$message.success('更新用户数据成功')
        }
      })
    },
    // 删除角色
    async removeRolesById (id) {
    //   询问是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      //   用户取消返回cancel,确认confirm
      if (confirmResult === 'cancel') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      } else {
        this.$message.success('删除角色成功！')
        this.getRolesList()
      }
    },
    // 删除权限
    async removeRightById (role, rightId) {
    //   询问是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      //   用户取消返回cancel,确认confirm
      if (confirmResult === 'cancel') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      //   role.chlidren = res.data
      role.children = res.data
    },
    // 显示分配权限对话框
    async showSetRightDialog (role) {
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      } else {
        this.rightslist = res.data
        this.getLeafKeys(role, this.defKeys)
        this.setRightDialogVisible = true
      }
      this.roleId = role.id
    },
    // 关闭对话框重置defkeys数组
    setRightDialogClose () {
      this.defKeys = []
    },
    // 通过递归获取三级权限
    getLeafKeys (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(element => {
        this.getLeafKeys(element, arr)
      })
    },
    // 提交角色权限到服务器
    async allotRights () {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      } else {
        this.$message.success(res.meta.msg)
        this.getRolesList()
        this.setRightDialogVisible = false
      }
    }
  }

}
</script>
<style lang="less" scoped>
.el-tag{
  margin: 7px;
}
.bdtop{
border-top: 1px solid #eeeeee;
}
.bdbottom{
border-bottom: 1px solid #eeeeee;
}
.vcenter{
    display: flex;
    align-items: center;
}

</style>
