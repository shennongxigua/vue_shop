<template>
  <div>
    <el-button @click="logout" type="info">退出</el-button>
  </div>
</template>

<script>
export default {
  methods: {
    // 退出登录:
    // 1. 清除token  2. 导航到登录页面
    logout() {      
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less" scoped>
</style>
