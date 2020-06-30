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
      <el-row class="addbtn">
        <el-col>
          <el-button type="primary"
                     @click="addCate">添加分类</el-button>
        </el-col>
      </el-row>

      <!-- 商品分类表格 -->
      <tableTree :data="catelist"
                 :columns="columns"
                 :show-index="true"
                 index-text=""
                 :show-row-hover="false"
                 :selection-type="false"
                 :expand-type="false"
                 border>
        <!-- 是否有效模板列 -->
        <template slot="isOk"
                  slot-scope="scope">
          <i class="el-icon-success"
             style="color:green"
             v-if="scope.row.cat_deleted===false"></i>
          <i class="el-icon-error"
             style="color:red"
             v-else></i>
        </template>
        <!-- 分类等级模板列 -->
        <template slot="order"
                  slot-scope="scope">
          <el-tag v-if="scope.row.cat_level===0">一级</el-tag>
          <el-tag type="success"
                  v-else-if="scope.row.cat_level===1">二级</el-tag>
          <el-tag type="warning"
                  v-else>三级</el-tag>
        </template>
        <!-- 分类等级模板列 -->
        <template slot="opt">
          <el-button type="primary"
                     size="mini">编辑</el-button>
          <el-button type="danger"
                     size="mini">删除</el-button>
        </template>

      </tableTree>

      <!-- 分页 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[3, 5, 10, 15]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>

    </el-card>

    <!-- 添加分类dialog -->
    <el-dialog title="添加分类"
               :visible.sync="addCateVisible"
               width="40%"
               @close="addCateClose">
      <el-form :model="addCateForm"
               :rules="addCateFormRules"
               ref="addCateFormRef"
               label-width="100px">
        <el-form-item label="分类名称："
                      prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader expand-trigger="hover"
                       v-model="selectedKeys"
                       :options="parentCatelist"
                       :props="cascaderProps"
                       @change="handleChange"
                       clearable
                       placeholder="请选择"></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="addCateVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="updataCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      catelist: [],
      total: 0,
      // 获取商品分类列表的查询参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 控制加添分类dialog的显示与隐藏
      addCateVisible: false,
      parentCatelist: [],
      selectedKeys: [],
      // 级联选择器的配置文件（绑定属性）
      cascaderProps: {
        // 可选择任意级别的节点
        checkStrictly: true,
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      addCateForm: {
        cat_name: '',
        // 默认添加一级分类
        cat_pid: 0,
        // 默认添加一级分类
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]

      },

      // tableTree的属性绑定，用来规定每列的显示信息
      columns: [
        { label: '分类名称', prop: 'cat_name' },
        // 设置当前列为模板列，模板template的slot属性名称为isOk
        { label: '是否有效', type: 'template', template: 'isOk' },
        // 设置当前列为模板列，模板template的slot属性名称为order
        { label: '分类等级', type: 'template', template: 'order' },
        // 设置当前列为模板列，模板template的slot属性名称为opt
        { label: '操作', type: 'template', template: 'opt', width: '150px' }
      ]
    }
  },
  methods: {
    // 获取分类列表
    getCateList: async function () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      this.total = res.data.total
      this.catelist = res.data.result
    },
    // 处理分页size变化
    handleSizeChange: function (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 处理分页current变化
    handleCurrentChange: function (newNum) {
      this.queryInfo.pagenum = newNum
      this.getCateList()
    },
    // 点击添加分类按钮
    addCate: async function () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      this.parentCatelist = res.data
      this.addCateVisible = true
      console.log(this.parentCatelist)
    },
    // 当级联选择器发生变化时
    handleChange: function () {
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
      console.log(this.addCateForm)
    },
    // 上传新添加的Cate
    updataCate: function () {
      this.$refs.addCateFormRef.validate(async flag => {
        if (!flag) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败！')
        }
        console.log(res)
        this.$message.success('添加分类成功！')
        // 关闭添加分类dialog
        this.addCateVisible = false
        // 刷新分类列表
        this.getCateList()
      })
    },
    // 关闭添加分类dialog
    addCateClose: function () {
      // 重置添加分类表单
      this.$refs.addCateFormRef.resetFields()
      // 重置已选中父级节点
      this.selectedKeys = []
      // 默认添加一级分类
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    }
  },
  created () {
    this.getCateList()
  }
}
</script>

<style lang="less" scoped>
.addbtn {
  margin-bottom: 20px;
}
.el-pagination {
  margin-top: 20px;
}
.el-cascader {
  width: 100%;
}
</style>
