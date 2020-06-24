<template>
  <div>
    <el-button @click="logOut">退出</el-button>
  </div>
</template>

<script>
export default {
  methods: {
    logOut: function () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>

<style lang="less" scoped></style>
