<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card class="box-card">
      <el-table :data="rigthsList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="权限名称" prop="authName"></el-table-column>
        <el-table-column label="路径" prop="path"></el-table-column>
        <el-table-column label="权限等级" prop="level">
          <template v-slot="slotProps">
            <el-tag v-if="slotProps.row.level === '0'">一级</el-tag>
            <el-tag type="success" v-if="slotProps.row.level === '1'">二级</el-tag>
            <el-tag type="warning" v-if="slotProps.row.level === '2'">三级</el-tag>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 所有的权限列表
      rigthsList: []
    }
  },
  created() {
    // 获取所有的权限
    this.getRigthsList()
  },
  methods: {
    // 获取权限列表
    async getRigthsList() {
      const { data: res } = await this.$http.get('rights/list')
      if (res.meta.status !== 200) return this.$meeage.error('获取权限列表失败！')
      this.rigthsList = res.data
    }
  }
}
</script>

<style lang="less" scoped>

</style>
