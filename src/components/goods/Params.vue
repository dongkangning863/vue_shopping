<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 警告文本 -->
      <el-alert title="注意：只允许为第三级分类设置相关参数！"
                type="warning"
                show-icon
                :closable="false">
      </el-alert>
      <!-- 级联选择器 -->
      <el-row>
        <el-col>
          <span>请选择商品：</span>
          <el-cascader v-model="selectedKeys"
                       :options="catelist"
                       :props="cascaderProps"
                       @change="handleChange"></el-cascader>
        </el-col>
      </el-row>
      <!-- tab标签页 -->
      <el-tabs v-model="activeName"
               @tab-click="handleTabClick">
        <el-tab-pane label="动态参数"
                     name="many">
          <!-- 添加参数按钮 -->
          <el-button type="primary"
                     size="mini"
                     :disabled="isBtnDisabled"
                     @click="addDialogVisible=true">添加参数</el-button>
          <!-- 动态参数展示表格 -->
          <el-table :data="manyParamslist"
                    style="width:100%"
                    border
                    stripe>
            <!-- index列 -->
            <el-table-column type="index"></el-table-column>
            <!-- 扩展列 -->
            <el-table-column type="expand">
              <!-- 循环渲染出获取的tag标签 -->
              <template slot-scope="scope">
                <el-tag v-for="(item,index) in scope.row.attr_vals"
                        :key="index"
                        closable
                        @close="colseTag(index,scope.row)">{{item}}</el-tag>
                <!-- 添加新的tag标签  -->
                <el-input class="input-new-tag"
                          v-if="scope.row.inputVisible"
                          v-model="scope.row.inputValue"
                          ref="inputRef"
                          size="small"
                          @keyup.enter.native="handleInputConfirm(scope.row)"
                          @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else
                           class="button-new-tag"
                           size="small"
                           @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 参数名称列 -->
            <el-table-column label="参数名称"
                             prop="attr_name"></el-table-column>
            <!-- 操作列 -->
            <el-table-column label="操作"
                             width="200px">
              <template slot-scope="scope">
                <el-button type="primary"
                           icon="el-icon-edit"
                           size="mini"
                           @click="editParams(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger"
                           icon="el-icon-delete"
                           size="mini"
                           @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>

        <el-tab-pane label="静态属性"
                     name="only">
          <!-- 添加参数按钮 -->
          <el-button type="primary"
                     size="mini"
                     :disabled="isBtnDisabled"
                     @click="addDialogVisible=true">添加属性</el-button>
          <!-- 静态参数展示表格 -->
          <el-table :data="onlyParamslist"
                    style="width:100%"
                    border
                    stripe>
            <!-- index列 -->
            <el-table-column type="index"></el-table-column>
            <!-- 扩展列 -->
            <el-table-column type="expand">
              <!-- 循环渲染出获取的tag标签 -->
              <template slot-scope="scope">
                <el-tag v-for="(item,index) in scope.row.attr_vals"
                        :key="index"
                        closable
                        @close="colseTag(index,scope.row)">{{item}}</el-tag>
                <!-- 添加新的tag标签  -->
                <el-input class="input-new-tag"
                          v-if="scope.row.inputVisible"
                          v-model="scope.row.inputValue"
                          ref="inputRef"
                          size="small"
                          @keyup.enter.native="handleInputConfirm(scope.row)"
                          @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else
                           class="button-new-tag"
                           size="small"
                           @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 属性名称列 -->
            <el-table-column label="属性名称"
                             prop="attr_name"></el-table-column>
            <!-- 操作列 -->
            <el-table-column label="操作"
                             width="200px">
              <template slot-scope="scope">
                <el-button type="primary"
                           icon="el-icon-edit"
                           size="mini"
                           @click="editParams(scope.row.attr_id)">编辑</el-button>
                <el-button type="danger"
                           icon="el-icon-delete"
                           size="mini"
                           @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数/属性的dialog -->
    <el-dialog :title="'添加'+textTitle"
               :visible.sync="addDialogVisible"
               width="50%">
      <!-- 将要提交的form表单 -->
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addFormRef"
               label-width="100px">
        <el-form-item :label="textTitle"
                      prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑参数/属性的dialog -->
    <el-dialog :title="'添加'+textTitle"
               :visible.sync="editDialogVisible"
               width="50%">
      <!-- 将要提交的form表单 -->
      <el-form :model="editForm"
               :rules="editFormRules"
               ref="editFormRef"
               label-width="100px">
        <el-form-item :label="textTitle"
                      prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="updateParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      catelist: [],
      selectedKeys: [],
      // cascader的配置文件（属性绑定）
      cascaderProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      // 默认激活的tab标签
      activeName: 'many',
      // 动态参数列表
      manyParamslist: [],
      // 静态参数列表
      onlyParamslist: [],
      // 控制dialog的显示
      addDialogVisible: false,
      editDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [
          { required: true, message: '请输入添加内容', trigger: 'blur' }
        ]
      },
      editForm: {
        attr_name: ''
      },
      editFormRules: {
        attr_name: [
          { required: true, message: '请输入内容', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    // 获取商品分类列表
    getCatelist: async function () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品列表失败！')
      }
      this.catelist = res.data
    },
    // 当级联选择器取值改变时
    handleChange: function () {
      // 判断选中的是否是三级分类
      if (this.selectedKeys.length !== 3) {
        this.selectedKeys = []
        this.manyParamslist = []
        this.onlyParamslist = []
        return
      }
      this.getParamslist()
    },
    // 当tab标签切换时
    handleTabClick: function () {
      this.getParamslist()
    },
    // 获取商品参数列表
    getParamslist: async function () {
      const { data: res } = await this.$http.get('categories/' + this.cateId + '/attributes', { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        this.$message.error('获取商品参数列表失败！')
      }
      // 把res.data中attr_vals属性的值由字符串转换为数组
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals.split(' ')
        // 控制new tag标签显示切换，默认显示为button
        item.inputVisible = false
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyParamslist = res.data
      } else {
        this.onlyParamslist = res.data
      }
    },
    // 添加新的参数/属性
    addParams: function () {
      this.$refs.addFormRef.validate(async flag => {
        if (!flag) return
        const { data: res } = await this.$http.post('categories/' + this.cateId + '/attributes', {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加属性失败')
        }
        this.$message.success('添加属性成功！')
        // 关闭dialog
        this.addDialogVisible = false
        // 刷新参数列表
        this.getParamslist()
      })
    },
    // 编辑参数/属性
    editParams: async function (id) {
      const { data: res } = await this.$http.get('categories/' + this.cateId + '/attributes/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取该商品的信息失败！')
      }
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 上传更新参数/属性
    updateParams: function () {
      this.$refs.editFormRef.validate(async flag => {
        const { data: res } = await this.$http.put('categories/' + this.cateId + '/attributes/' + this.editForm.attr_id, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) {
          return this.$message.error('更新商品信息失败！')
        }
        this.$message.success('更新商品信息成功！')
        // 关闭编辑dialog
        this.editDialogVisible = false
        // 刷新商品参数/属性列表
        this.getParamslist()
      })
    },
    // 删除参数/属性
    removeParams: async function (id) {
      const confirmText = await this.$confirm('此操作将永久删除该属性, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (confirmText !== 'confirm') {
        return this.$message.info('已取消该操作')
      }
      const { data: res } = await this.$http.delete('categories/' + this.cateId + '/attributes/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除操作失败！')
      }
      this.$message.success('删除成功！')
      // 刷新属性列表
      this.getParamslist()
    },
    // 点击新增tag标签时
    showInput: function (row) {
      // 切换到input展示
      row.inputVisible = true
      // input自动获取焦点
      // nextTick是让代码在元素加载完毕后调用
      this.$nextTick(_ => {
        this.$refs.inputRef.$refs.input.focus()
      })
    },
    // 处理新增tag标签input失去焦点时
    handleInputConfirm: async function (row) {
      // 验证输入的内容是否合法
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue)
      row.inputValue = ''
      // 保存attr_vals数组中的内容到服务器，持久化保存到数据库中
      this.saveAttr_vals(row)
    },
    // 保存前端对于attr_vals的处理
    saveAttr_vals: async function (row) {
      const { data: res } = await this.$http.put('categories/' + this.cateId + '/attributes/' + row.attr_id, {
        attr_name: row.attr_name,
        attr_sel: this.activeName,
        attr_vals: row.attr_vals.join(' ')
      })
      if (res.meta.status !== 200) {
        return this.$message.error('修改属性项失败！')
      }
      this.$message.success('修改属性项成功')
      // 切换到button显示
      row.inputVisible = false
    },
    colseTag: function (index, row) {
      // 删除attr_vals数组中指定索引的元素
      row.attr_vals.splice(index, 1)
      // 保存attr_vals数组中的内容到服务器，持久化保存到数据库中
      this.saveAttr_vals(row)
    }
  },
  computed: {
    isBtnDisabled () {
      if (this.selectedKeys.length === 3) {
        return false
      } else {
        return true
      }
    },
    cateId () {
      if (this.selectedKeys.length === 3) {
        return this.selectedKeys[2]
      } else {
        return null
      }
    },
    textTitle () {
      if (this.activeName === 'many') {
        return '新的参数'
      } else {
        return '新的属性'
      }
    }
  },
  created () {
    this.getCatelist()
  }
}
</script>

<style lang="less" scoped>
.el-alert {
  margin-bottom: 20px;
}
.el-tabs {
  margin-top: 20px;
  .el-table {
    margin-top: 15px;
    .el-input {
      width: 150px;
    }
  }
}
.el-tag {
  margin-right: 20px;
}
</style>
