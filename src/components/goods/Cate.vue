<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card class="box-card">
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <tree-table :data="catelist" :columns="columns" :selection-type="false"
       show-index index-text="#" border class="treeTbale">
        <!-- 是否有效 -->
        <template v-slot:isok="slotProps">
          <i class="el-icon-success" v-if="slotProps.row.cat_deleted === false"
           style="color: lightgreen; font-size: 24px;"></i>
          <i class="el-icon-error" v-else style="color: red; font-size"></i>
        </template>
        <!-- 排序 -->
        <template v-slot:order="slotProps">
          <el-tag v-if="slotProps.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" v-else-if="slotProps.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" v-else-if="slotProps.row.cat_level === 2">三级</el-tag>
        </template>
        <!-- 操作 -->
        <template v-slot:opt="slotProps">
          <el-button type="primary" icon="el-icon-edit" size="mini" @click="set(slotProps.row)">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
          :current-page="queryInfo.pagenum" :page-sizes="[3, 5, 10]"
          :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper"
          :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加分类的对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
      <!-- 内容主体区 -->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="80px">
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <!-- 级联选择器 -->
          <el-cascader v-model="selectedKeys" :options="parentCateList"
            :props="cascaderProps" @change="parentCateChange" clearable change-on-select></el-cascader>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表
      catelist: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        { label: '分类名称', prop: 'cat_name', minWidth: '170px' },
        { label: '是否有效', type: 'template', template: 'isok' },
        { label: '排序', type: 'template', template: 'order' },
        { label: '操作', type: 'template', template: 'opt', minWidth: '180px' }
      ],
      // 控制添加分类对话框的显示与隐藏
      addCateDialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
        // 分类名称
        cat_name: '',
        // 分类父 ID
        cat_pid: 0,
        // 分类层级 默认一级分类
        cat_level: 0
      },
      // 添加分类表单验证对象
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名', trigger: 'blur' }
        ]
      },
      // 父级分类列表
      parentCateList: [],
      // 级联选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      // 选中的父级分类id数组
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('/categories', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) return this.$message.error('获取商品分类失败!')
      this.catelist = res.data.result
      this.total = res.data.total
    },
    // 监听pagesize 的改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum的改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮展示添加分类的对话框
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 添加分类对话框关闭时触发
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('/categories', { params: { type: 2 } })
      if (res.meta.status !== 200) return this.$message.error('获取父级商品分类失败!')
      this.parentCateList = res.data
    },
    // 选择项发生变化时触发
    parentCateChange() {
      if (this.selectedKeys.length > 0) {
        // 为添加分类表单更新数据
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 提交添加分类表单
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) this.$message.error('添加分类失败')
        this.$message.success('添加分类成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    }
  }
}
</script>

<style lang="less" scoped>
.treeTbale {
  margin-top: 15px;
}
</style>
