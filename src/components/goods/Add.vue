<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card class="box-card">
    <el-alert title="添加" type="info" :closable="false" center show-icon></el-alert>
    <!-- 步骤条区域 -->
    <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
      <el-step title="基本信息"></el-step>
      <el-step title="商品参数"></el-step>
      <el-step title="商品属性"></el-step>
      <el-step title="商品图片"></el-step>
      <el-step title="商品内容"></el-step>
      <el-step title="完成"></el-step>
    </el-steps>
    <!-- 添加商品的表单 -->
    <el-form :model="addForm" :rules="addFromRules" ref="addFormRef" label-width="100px" label-position="top">
      <!-- 左侧Tabs标签 -->
      <el-tabs v-model="activeIndex" tab-position="left" :before-leave="beforeTabLeave" @tab-click="tabClicked">
        <el-tab-pane label="基本信息" name="0">
          <el-form-item label="商品名称" prop="goods_name">
            <el-input v-model="addForm.goods_name"></el-input>
          </el-form-item>
          <el-form-item label="商品价格" prop="goods_price">
            <el-input v-model="addForm.goods_price" type="number"></el-input>
          </el-form-item>
          <el-form-item label="商品重量" prop="goods_weight">
            <el-input v-model="addForm.goods_weight" type="number"></el-input>
          </el-form-item>
          <el-form-item label="商品数量" prop="goods_number">
            <el-input v-model="addForm.goods_number" type="number"></el-input>
          </el-form-item>
          <el-form-item label="商品分类" prop="goods_cat">
            <el-cascader v-model="addForm.goods_cat" :options="catelist" :props="cateProps"
              @change="handleChange"></el-cascader>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品参数" name="1">
          <!-- 渲染表单的item项 -->
          <el-form-item :label="item.attr_name" v-for="item in manyTableData" :key="item.attr_id">
            <!-- 复选框组 -->
            <el-checkbox-group v-model="item.attr_vals">
              <el-checkbox :label="cb" v-for="(cb, i) in item.attr_vals" :key="i" border size="mini"></el-checkbox>
            </el-checkbox-group>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品属性" name="2">
          <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
            <el-input v-model="item.attr_vals"></el-input>
          </el-form-item>
        </el-tab-pane>
        <el-tab-pane label="商品图片" name="3">
          <!-- action图片上传到的后台api地址  -->
          <el-upload :action="uploadURL" :on-preview="handlePreview" :on-remove="handleRemove"
           :on-success="handleSuccess" list-type="picture" :headers="headerObj" >
            <el-button size="small" type="primary">点击上传</el-button>
          </el-upload>
        </el-tab-pane>
        <el-tab-pane label="商品内容" name="4">
          <!-- 富文本编辑器 -->
          <quill-editor v-model="addForm.goods_introduce">
          </quill-editor>
          <el-button type="primary" class="btnAdd" @click="add">添加商品</el-button>
        </el-tab-pane>
      </el-tabs>
    </el-form>
    </el-card>
    <!-- 图片预览对话框 -->
    <el-dialog  title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="previewPath" alt="" class="previewImg">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  data() {
    return {
      // 进度条激活的索引
      activeIndex: '0',
      // 添加商品的表单数据对象
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        // 商品所属的分类数组
        goods_cat: [],
        // 图片的数组
        pics: [],
        // 商品详情描述
        goods_introduce: '',
        attrs: []
      },
      // 添加商品名称的数据验证对象
      addFromRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', tirgger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', tirgger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', tirgger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', tirgger: 'blur' }
        ],
        goods_cat: [
          { type: 'array', required: true, message: '请选中商品分类', tirgger: 'blur' }
        ]
      },
      // 商品分类数据对象
      catelist: [],
      // 练级选择器配置对象
      cateProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      // 动态参数列表
      manyTableData: [],
      // 静态属性列表
      onlyTableData: [],
      // 图片上传的rul地址
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 为上传图片设置请求头
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewPath: '',
      // 控住预览图片对话框的显示与隐藏
      previewVisible: false
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取商品分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) return this.$message.error('获取商品分类数据失败！')
      this.catelist = res.data
    },
    // 监听级联选择器选择项变化
    handleChange() {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    // tab页被点击时触发
    async tabClicked() {
      // 证明访问的时动态参数面板
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: { sel: 'many' }
        })
        if (res.meta.status !== 200) return this.$message.error('获取动态参数失败！')
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(',')
        })
        this.manyTableData = res.data
        console.log(this.manyTableData)
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
          params: { sel: 'only' }
        })
        if (res.meta.status !== 200) return this.$message.error('获取静态属性失败！')
        this.onlyTableData = res.data
      }
    },
    // tab页切换时触发
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请选择商品分类')
        return false
      }
    },
    // 处理上传图片预览
    handlePreview(file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 处理移除图片的操作
    handleRemove(file) {
      // 1. 获取要删除图片的临时路径
      const filePath = file.response.data.tep_path
      // 2. 在pics数组中导致图片路径对应的索引值
      const i = this.addForm.pics.findIndex(item => {
        item.pic = filePath
      })
      // 3. 调用splice 方法删除图片信息
      this.addForm.pics.splice(i)
    },
    // 监听图片上传成功
    handleSuccess(response) {
      // 1. 拼接一个图片信息对象
      const prcInfo = { pic: response.data.tmp_path }
      // 2. 将图片信息对象push到 pics数组中
      this.addForm.pics.push(prcInfo)
    },
    // 添加商品
    add() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return this.$message.error('请填写必要的表单项！')
        // 执行添加业务逻辑
        // lodash  cloneDeep(obj)
        // 将goods_cat数组中的数据转化为 , 分割的字符串
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals.join(' ') }
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        // 发起请求添加商品
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) return this.$message.error('添加商品失败！')
        this.$message.success('添加商品成功！')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-checkbox {
  margin: 0 0 0 10px;
}
.previewImg {
  width: 100%;
}
.btnAdd {
  margin-top: 15px;
}
</style>
