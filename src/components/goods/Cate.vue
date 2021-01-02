<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加分类按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>
      <!-- 表格   -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        border
        :show-row-hover="false"
        class="treeTable"
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            size="mini"
            type="success"
            v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="showeditCateDialog(scope.row)"
            >编辑</el-button
          >
          <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCateById(scope.row.cat_id)"
            >删除</el-button
          >
        </template>
      </tree-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="querInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClose"
    >
      <!-- 内容主体 -->
      <el-form
        :model="addCateFrom"
        :rules="addCateFromRules"
        ref="addCateFromRef"
        label-width="100px"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateFrom.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <!-- 下方区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑分类对话框 -->
    <el-dialog
      title="编辑分类"
      :visible.sync="editCateDialogVisible"
      width="50%"
      @close="editCateDialogClose"
    >
      <!-- 内容主体 -->
      <el-form
        :model="editCateFrom"
        :rules="editCateFromRules"
        ref="editCateFromRef"
        label-width="100px"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="editCateFrom.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <!-- 下方区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 商品分类数据
      cateList: [],
      // 查询条件
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      //   总数居条数
      total: 0,
      addCateDialogVisible: false,
      //   父级分类列表
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        checkStrictly: true,
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类的Id数组
      selectedKeys: [],
      //   添加分类
      addCateFrom: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateFromRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 编辑分类
      editCateDialogVisible: false,
      editCateFrom: {
        cat_id: 0,
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      editCateFromRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]

      },
      //   为table指定列定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          type: 'template',
          template: 'isok'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ]
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: this.querInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.cateList = res.data.result
      this.total = res.data.total
    },
    handleSizeChange (newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    // 显示添加用户对话框
    showAddCateDialog () {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类节点
    async getParentCateList () {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.status)
      }
      this.parentCateList = res.data
    },
    // 选择性发生变化触发
    parentCateChanged () {
      if (this.selectedKeys.length >= 0) {
        //   父级分类id值
        this.addCateFrom.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateFrom.cat_level = this.selectedKeys.length
      } else {
        this.addCateFrom.cat_pid = 0
        this.addCateFrom.cat_level = 0
      }
    },
    // 关闭重置
    addCateDialogClose () {
      this.$refs.addCateFromRef.resetFields()
      this.selectedKeys = []
      this.addCateFrom.cat_level = 0
      this.addCateFrom.cat_pid = 0
    },
    // 点击按钮添加分类
    addCate () {
      this.$refs.addCateFromRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateFrom)
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.msg)
        } else {
          this.$message.success('添加分类成功1')
          this.getCateList()
          this.addCateDialogVisible = false
        }
      })
    },
    // 显示分类
    showeditCateDialog (info) {
      this.editCateFrom.cat_name = info.cat_name
      this.editCateFrom.cat_id = info.cat_id
      this.editCateDialogVisible = true
    },
    // 重置
    editCateDialogClose () {
      this.$refs.editCateFromRef.resetFields()
      this.editCateFrom.cat_level = 0
      this.editCateFrom.cat_pid = 0
      this.editCateFrom.cat_name = ''
      this.editCateFrom.cat_id = 0
    },
    // 编辑分类
    async editCate () {
      const { data: res } = await this.$http.put('categories/' + this.editCateFrom.cat_id, { cat_name: this.editCateFrom.cat_name })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      } else {
        this.$message.success('更新分类成功!')
        this.getCateList()
        this.editCateDialogVisible = false
      }
    },
    // 根据id删除分类
    async removeCateById (id) {
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
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      } else {
        this.$message.success('删除分类成功！')
        this.getCateList()
      }
    }
  }
}
</script>
<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader{
    width: 100%;
}

</style>
