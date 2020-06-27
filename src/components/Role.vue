<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table :data="roleList"
                border
                stripe>
        <!-- 角色权限下拉菜单 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <div>
              <el-row v-for="(item1,index1) in scope.row.children"
                      :key="item1.id"
                      :class="['bdbottom',index1===0?'bdtop':'','vcenter']">
                <!-- 渲染一级权限 -->
                <el-col :span="5">
                  <el-tag>{{item1.authName}}</el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <!-- 渲染二级、三级权限 -->
                <el-col :span="19">
                  <el-row v-for="(item2,index2) in item1.children"
                          :key="item2.id"
                          :class="[index2===0?'':'bdtop','vcenter']">
                    <!-- 渲染二级权限 -->
                    <el-col :span="6">
                      <el-tag type="success">{{item2.authName}}</el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <!-- 渲染三级权限 -->
                    <el-col :span="18">
                      <el-tag v-for="item3 in item2.children"
                              :key="item3.id"
                              type="warning"
                              closable
                              @close="removeRights(scope.row,item3.id)">{{item3.authName}}</el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </div>
          </template>
        </el-table-column>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="roleName"
                         label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc"
                         label="角色简述"></el-table-column>
        <el-table-column label="权限等级"
                         width="300px">
          <template slot-scope="scope">
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini">编辑</el-button>
            <el-button type="danger"
                       icon="el-icon-delete"
                       size="mini">删除</el-button>
            <el-button type="warning"
                       icon="el-icon-setting"
                       size="mini"
                       @click="setRights(scope.row)">分配权限</el-button>

          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限dialog -->
    <el-dialog title="权限分配"
               :visible.sync="setRightsDialogVisible"
               width="40%"
               @close="setRightsDialogClosed">
      <!-- 树形结构 -->
      <el-tree :data="rightsList"
               :props="treeProps"
               show-checkbox
               node-key="id"
               default-expand-all
               :default-checked-keys="checkedKeys"
               ref="treeRef"></el-tree>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="setRightsDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="updateRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 角色列表
      roleList: [],
      // 控制权限分配dialog的显示
      setRightsDialogVisible: false,
      // 权限列表
      rightsList: [],
      // 树形结构属性绑定
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      // 默认选择的节点
      checkedKeys: [],
      roleId: ''
    }
  },
  methods: {
    // 获取角色列表
    getRoleList: async function () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败！')
      }
      this.roleList = res.data
    },
    // 删除id所对应的权限
    removeRights: async function (role, id) {
      const removeConfirm = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (removeConfirm === 'cancel') {
        return this.$message.info('已取消该操作！')
      }
      if (removeConfirm === 'confirm') {
        const { data: res } = await this.$http.delete('roles/' + role.id + '/rights/' + id)
        if (res.meta.status !== 200) {
          return this.$message.error('删除权限失败！')
        }
        console.log(role)
        // res.data返回的是删除后的权限数据
        role.children = res.data
      }
    },
    // 权限分配按钮
    setRights: async function (role) {
      // 保存角色id在后续提交权限分配的时候会用到
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      this.rightsList = res.data
      this.getKeys(role, this.checkedKeys)
      this.setRightsDialogVisible = true
    },
    // 利用递归,循环获取所有三级节点的id
    getKeys: function (node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => {
        this.getKeys(item, arr)
      })
    },
    setRightsDialogClosed: function () {
      // 关闭分配权限dialog后清空保存默认选择节点的数组
      this.checkedKeys = []
    },
    // 上传更新权限数据
    updateRights: async function () {
      const idArr = [...this.$refs.treeRef.getCheckedKeys(), ...this.$refs.treeRef.getHalfCheckedKeys()]
      const idStr = idArr.join(',')
      const { data: res } = await this.$http.post('roles/' + this.roleId + '/rights', { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('设置权限失败！')
      }
      this.$message.success('设置权限成功！')
      // 关闭dialog
      this.setRightsDialogVisible = false
      // 刷新角色列表
      this.getRoleList()
    }
  },
  created () {
    this.getRoleList()
  }
}
</script>

<style lang="less" scoped>
.el-table {
  margin-top: 15px;
}
.el-tag {
  margin: 10px 0 10px 20px;
}
i {
  margin-left: 5px;
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
