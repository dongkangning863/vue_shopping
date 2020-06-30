<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 搜索区域 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容"
                    v-model="queryInfo.query"
                    clearable
                    @clear="getGoodslist">
            <el-button slot="append"
                       icon="el-icon-search"
                       @click="getGoodslist"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary"
                     @click="toAddGoods">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 商品列表区域 -->
      <el-table :data="goodslist"
                border
                stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名称"
                         prop="goods_name"></el-table-column>
        <el-table-column label="商品价格"
                         prop="goods_price"
                         width="80px"></el-table-column>
        <el-table-column label="商品重量"
                         prop="goods_weight"
                         width="80px"></el-table-column>
        <el-table-column label="创建时间"
                         width="170px">
          <template slot-scope="scope">
            {{scope.row.add_time|dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作"
                         width="130px">
          <template slot-scope="scope">
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini"></el-button>
            <el-button type="danger"
                       icon="el-icon-delete"
                       size="mini"
                       @click="removeGoods(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination @size-change="handleSizeChange"
                     @current-change="handleCurrentChange"
                     :current-page="queryInfo.pagenum"
                     :page-sizes="[5, 10, 15, 20]"
                     :page-size="queryInfo.pagesize"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total">
      </el-pagination>
    </el-card>
  </div>
</template>
<script>
export default {
  data () {
    return {
      // 商品列表
      goodslist: [],
      // 商品总数目
      total: 0,
      // 获取商品列表的请求参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      }
    }
  },
  methods: {
    getGoodslist: async function () {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败！')
      }
      this.goodslist = res.data.goods
      this.total = res.data.total
      console.log(res)
    },
    handleSizeChange: function (newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodslist()
    },
    handleCurrentChange: function (newCurrent) {
      this.queryInfo.pagenum = newCurrent
      this.getGoodslist()
    },
    removeGoods: async function (id) {
      const { data: res } = await this.$http.delete('goods/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除商品失败')
      }
      this.$message.success('删除商品成功')
      this.getGoodslist()
    },
    toAddGoods: function () {
      this.$router.push('/goods/add')
    }
  },
  created () {
    this.getGoodslist()
  }
}
</script>
<style lang="less" scoped>
.el-col {
  margin-right: 20px;
}
.el-table {
  margin-top: 20px;
}
</style>
