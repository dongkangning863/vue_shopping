<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 提示 -->
      <el-alert title="添加商品信息"
                type="info"
                center
                show-icon
                :closable="false">
      </el-alert>
      <!-- 步骤条 -->
      <el-steps :space="200"
                :active="activeIndex-0"
                finish-status="success"
                align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- tabs标签页 -->
      <!-- 把tabs用form表单包裹起来，方便之后向后端提交数据 -->
      <el-form :model="addForm"
               :rules="addFormRules"
               ref="addFormRef"
               label-width="100px"
               label-position="top">
        <el-tabs tab-position="left"
                 v-model="activeIndex"
                 :before-leave="beforeLeave"
                 @tab-click="tabClick">
          <el-tab-pane label="基本信息"
                       name="0">
            <el-form-item label="商品名称"
                          prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格"
                          prop="goods_price">
              <el-input v-model="addForm.goods_price"></el-input>
            </el-form-item>
            <el-form-item label="商品重量"
                          prop="goods_weight">
              <el-input v-model="addForm.goods_weight"></el-input>
            </el-form-item>
            <el-form-item label="商品数量"
                          prop="goods_number">
              <el-input v-model="addForm.goods_number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类"
                          prop="goods_cat">
              <el-cascader v-model="addForm.goods_cat"
                           :options="catelist"
                           :props="props"
                           @change="handleChange"></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数"
                       name="1">
            <el-form-item :label="item1.attr_name"
                          v-for="item1 in manylist"
                          :key="item1.attr_id">
              <el-checkbox-group v-model="item1.attr_vals">
                <el-checkbox :label="item2"
                             v-for="(item2,index) in item1.attr_vals"
                             :key="index"
                             border></el-checkbox>
              </el-checkbox-group>
            </el-form-item>

          </el-tab-pane>
          <el-tab-pane label="商品属性"
                       name="2">
            <el-form-item :label="item.attr_name"
                          v-for="item in onlylist"
                          :key="item.attr_id">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片"
                       name="3">
            <!--  文件上传 -->
            <!-- action指定上传地址(绝对路径)-->
            <!-- 因为上传请求并非axios，所以请求头中没有包含设定好的token，需要自己设置请求头 -->
            <el-upload action="http://127.0.0.1:8888/api/private/v1/upload"
                       :on-preview="handlePreview"
                       :on-remove="handleRemove"
                       :headers="uploadHeaders"
                       :on-success="uploadSuccess"
                       list-type="picture">
              <el-button size="small"
                         type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容"
                       name="4">
            <quill-editor v-model="addForm.goods_introduce">
            </quill-editor>
            <el-button type="primary"
                       class="addbtn"
                       @click="addGoods">添加商品</el-button>
          </el-tab-pane>
          <el-tab-pane label="完成"
                       name="5">完成</el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <!-- 图片预览dialog -->
    <el-dialog title="图片预览"
               :visible.sync="previewVisible"
               width="40%">
      <el-image style="width: 100%"
                :src="previewURL"
                fit="contain"></el-image>
    </el-dialog>
  </div>
</template>
<script>
import _ from 'lodash'
export default {
  data () {
    return {
      activeIndex: '',
      catelist: [],
      addForm: {
        goods_name: '',
        goods_price: 0,
        goods_weight: 0,
        goods_number: 0,
        goods_cat: [],
        pics: [],
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' }
        ]
      },
      props: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      manylist: [],
      onlylist: [],
      uploadHeaders: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewURL: '',
      previewVisible: false
    }
  },
  methods: {
    // 获取商品分类数据
    getcatelist: async function () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类列表失败！')
      }
      this.catelist = res.data
    },
    // 当级联选择器取值发生变化时
    handleChange: function () {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    // 控制tags标签页切换
    beforeLeave: function (activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    // tabs标签页切换时
    tabClick: async function () {
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get('categories/' + this.cateId + '/attributes', { params: { sel: 'many' } })
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品动态属性失败！')
        }
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manylist = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get('categories/' + this.cateId + '/attributes', { params: { sel: 'only' } })
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品动态属性失败！')
        }
        this.onlylist = res.data
      }
    },
    // 当触发图片预览时
    handlePreview: function (file) {
      this.previewURL = file.response.data.url
      this.previewVisible = true
    },
    // 当触发移除图片时
    handleRemove: function (file) {
      console.log(file)
      const index = this.addForm.pics.findIndex(item =>
        item.pic === file.response.data.tmp_path
      )
      this.addForm.pics.splice(index, 1)
    },
    // 图片上传成功
    uploadSuccess: function (response) {
      const picObj = {
        pic: response.data.tmp_path
      }
      this.addForm.pics.push(picObj)
    },
    // 添加商品
    addGoods: function () {
      // 装载动态参数到attrs中
      this.manylist.forEach(item => {
        const attrObj = {
          attr_id: item.attr_id,
          attr_value: item.attr_vals.join(' ')
        }
        this.addForm.attrs.push(attrObj)
      })
      // 装载静态属性到attrs中
      this.onlylist.forEach(item => {
        const attrObj = {
          attr_id: item.attr_id,
          attr_value: item.attr_vals
        }
        this.addForm.attrs.push(attrObj)
      })
      // 克隆addform（防止直接操作addform，因为addform与前端ui绑定直接操作可能会报错）
      const form = _.cloneDeep(this.addForm)
      form.goods_cat = form.goods_cat.join(',')
      console.log(form)
      this.$refs.addFormRef.validate(async flag => {
        if (!flag) {
          return this.$message.error('请填写完整商品信息')
        }
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) {
          return this.$message.error('添加商品失败')
        }
        this.$message.success('添加商品成功!')
        // 重置addform表单
        this.$refs.addFormRef.resetFields()
        this.addForm.attrs = []
        this.addForm.pics = []
        this.addForm.goods_introduce = ''
        // 跳转到商品列表页面
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId: function () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      } else {
        return null
      }
    }
  },
  created () {
    this.getcatelist()
  }
}
</script>
<style lang="less" scoped>
.el-steps {
  margin: 20px 0;
}
.el-checkbox {
  margin-right: 10px;
}
.addbtn {
  margin-top: 80px;
}
</style>
