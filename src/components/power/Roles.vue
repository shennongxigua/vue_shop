<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card class="box-card">
      <el-row>
        <el-col>
          <el-button type="primary" @click="addRoleDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表展示 -->
      <el-table :data="rolelist" stripe border>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template v-slot='slotProps'>
            <!-- 添加角色列表区 -->
            <el-row :class="['bdbottom',i1 == 0 ? 'bdtop':'','vcenter']" v-for="(item1, i1) in slotProps.row.children" :key="item1.id">
              <!-- 渲染一级权限 -->
              <el-col :span="6">
                <el-tag closable @close="removeRightByid(slotProps.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-arrow-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="18">
                <!-- 通过循环嵌套渲染 -->
                <el-row :class="[i2 === 0 ? '':'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                  <el-col :span="8">
                    <el-tag type="success" closable @close="removeRightByid(slotProps.row,item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-arrow-right"></i>
                  </el-col>
                  <el-col :span="16">
                    <el-tag :class="[i3 === 0 ? '':'bdtop']" type="warning" v-for="(item3,i3) in item2.children"
                      :key="item3.id" closable @close="removeRightByid(slotProps.row,item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" prop width="300px">
          <template v-slot="slotProps">
            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDailog(slotProps.row.id)">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(slotProps.row.id)">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(slotProps.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色的对话框 -->
    <el-dialog title="添加新角色" :visible.sync="addRoleDialogVisible" width="50%" @close="addRoleDialogClosed">
      <!-- 内容主体区 -->
      <el-form :model="roleForm" :rules="addRoleFormRules" ref="addRoleFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="roleForm.roleName" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="roleForm.roleDesc" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改角色的对话框 -->
    <el-dialog title="修改角色" :visible.sync="editRoleDialogVisible" width="50%" @close="editRoleDialogClosed">
      <!-- 内容主体区 -->
      <el-form :model="editRoleForm" :rules="editRoleFormRules" ref="editRoleFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editRoleForm.roleName" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editRoleForm.roleDesc" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="editRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRole">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%">
      <!-- 树型控件-->
      <el-tree :data="rightslist" :props="treeProps" node-key="id" ref="treeRef"
       show-checkbox default-expand-all :default-checked-keys="defKeys" ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有角色数据
      rolelist: [],
      // 控住添加角色对话框显示和隐藏
      addRoleDialogVisible: false,
      // 修改角色对话框的显示与隐藏
      editRoleDialogVisible: false,
      // 分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 添加角色表单数据
      roleForm: {
        roleName: '',
        roleDesc: ''
      },
      // 修改角色表单数据
      editRoleForm: {
        roleId: '',
        roleName: '',
        roleDesc: ''
      },
      // 树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 所有权限的数据
      rightslist: [],
      // 默认选择的的几点id值数组
      defKeys: [],
      // 添加角色表单的验证规则
      addRoleFormRules: {
        roleName: [{ required: true, message: '请输入角色名', trigger: 'blur' }]
      },
      // 修改角色表单的验证
      editRoleFormRules: {
        roleName: [{ required: true, message: '请输入角色名', trigger: 'blur' }]
      },
      // 当前即将分配权限角色的id
      roleId: ''
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取数据失败')
      this.rolelist = res.data
    },
    // 提交添加角色表单
    addRole() {
      this.$refs.addRoleFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) return
        // 验证成功 发起请求
        const { data: res } = await this.$http.post('roles', this.roleForm)
        if (res.meta.status !== 201) return this.$message.error('添加角色失败！')
        this.$message.success('添加角色成功！')
        // 添加完成后关闭对话框
        this.addRoleDialogVisible = false
        // 刷新用户列表
        this.getRolesList()
      })
    },
    // 提交修改角色表单
    editRole() {
      this.$refs.editRoleFormRef.validate(async valid => {
        if (!valid) return
        // 验证成功 发起请求
        const { data: res } = await this.$http.put(`roles/${this.editRoleForm.roleId}`, { roleName: this.editRoleForm.roleName, roleDesc: this.editRoleForm.roleDesc })
        if (res.meta.status !== 200) return this.$message.error('修改角色信息失败！')
        this.$message.success('修改角色信息成功')
        this.editRoleDialogVisible = false
        this.getRolesList()
      })
    },
    // 监听添加角色对话框的关闭
    addRoleDialogClosed() {
      this.$refs.addRoleFormRef.resetFields()
    },
    // 修改角色对话框打开是获取数据
    async showEditDailog(id) {
      const { data: res } = await this.$http.get(`roles/${id}`)
      if (res.meta.status !== 200) return this.$message.error('查询用户信息失败！')
      this.editRoleForm = res.data
      this.editRoleDialogVisible = true
    },
    // 监听修改角色对话框的关闭
    editRoleDialogClosed() {
      this.$refs.editRoleFormRef.resetFields()
    },
    // 根据id删除对应的角色
    async removeRoleById(id) {
      console.log(id)
      // 利用confirmbox询问是否确定删除
      const confirmResult = await this.$confirm('此操作将永久删除这条记录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除返回字符串confirm
      // 如果用户取消删除返回字符串cancel
      if (confirmResult !== 'confirm') return this.$message.info('以取消删除！')
      // 确定删除 发起请求
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('服务器出错 删除失败!')
      this.$message.success('删除成功！')
      this.getRolesList()
    },
    // 根据id删除对应的权限
    async removeRightByid(role, rightId) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.info('取消删除！')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败！')
      this.$message.success('删除成功')
      // 利用返回的数据部分更新页面
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限失败！')
      this.rightslist = res.data
      // 递归获取三级节点的id
      this.defKeys = []
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 利用roleid获取左右三级节点的id保存到defKeys中 便于渲染树形节点
    getLeafKeys(node, arr) {
      // node节点中不包含children属性
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) return this.$message.error('分配权限失败！')
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
