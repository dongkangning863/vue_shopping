<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 搜索框 -->
      <el-row>
        <el-col :span="8">
          <el-input placeholder="请输入内容">
            <el-button slot="append"
                       icon="el-icon-search"></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 订单列表区域 -->
      <el-table :data="orderList"
                style="width: 100%"
                border
                stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="order_number"
                         label="订单编号">
        </el-table-column>
        <el-table-column prop="order_price"
                         label="订单价格"
                         width="80px">
        </el-table-column>
        <el-table-column prop="order_pay"
                         label="是否付款"
                         width="80px">
          <template slot-scope="scope">
            <el-tag type="success"
                    v-if="scope.row.order_pay==='1'">已付款</el-tag>
            <el-tag type="danger"
                    v-else>未付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="is_send"
                         label="是否发货"
                         width="80px">
        </el-table-column>
        <el-table-column label="下单时间"
                         width="200px">
          <template slot-scope="scope">
            {{scope.row.create_time|dateFormat}}
          </template>
        </el-table-column>
        <el-table-column label="操作"
                         width="180px">
          <template>
            <el-button type="primary"
                       icon="el-icon-edit"
                       size="mini"
                       @click="editDialogVisible=true">编辑</el-button>
            <el-button type="warning"
                       icon="el-icon-location"
                       size="mini"
                       @click="showLocationDialog">定位</el-button>
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
    <!-- 编辑地址dialog -->
    <el-dialog title="编辑地址"
               :visible.sync="editDialogVisible"
               width="50%"
               @close="closeEditDialog">
      <el-form :model="editForm"
               :rules="editFormRules"
               ref="editFormRef"
               label-width="100px">
        <el-form-item label="省市区/县"
                      prop="adress1">
          <el-cascader v-model="editForm.adress1"
                       :options="cityData"
                       :props="{ expandTrigger: 'hover' }"
                       style="width:100%"></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址"
                      prop="adress2">
          <el-input v-model="editForm.adress2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer"
            class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary"
                   @click="editDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 物流进度dialog -->
    <el-dialog title="编辑地址"
               :visible.sync="locationDialogVisible"
               width="50%">
      <el-timeline>
        <el-timeline-item v-for="(item, index) in locationData"
                          :key="index"
                          :timestamp="item.time">
          {{item.context}}
        </el-timeline-item>
      </el-timeline>
    </el-dialog>
  </div>
</template>
<script>
import cityData from './citydata.js'
export default {
  data () {
    return {
      orderList: [],
      total: 0,
      // 获取订单列表的请求参数
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      // 城市/区县数据
      cityData: cityData,
      editForm: {
        adress1: '',
        adress2: ''
      },
      editFormRules: {
        adress1: [
          { required: true, message: '请选择省市区/县', trigger: 'blur' }
        ],
        adress2: [
          { required: true, message: '请输入详细地址', trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      locationDialogVisible: false,
      locationData: []
    }
  },
  created () {
    this.getOrderlist()
  },
  methods: {
    getOrderlist: async function () {
      const { data: res } = await this.$http.get('orders', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败！')
      }
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    handleSizeChange: function (newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderlist()
    },
    handleCurrentChange: function (newCurrent) {
      this.queryInfo.pagenum = newCurrent
      this.getOrderlist()
    },
    closeEditDialog: function () {
      this.$refs.editFormRef.resetFields()
    },
    showLocationDialog: async function () {
      const { data: res } = await this.$http.get('/kuaidi/1106975712662')
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流信息失败！')
      }
      this.locationData = res.data
      this.locationDialogVisible = true
    }

  }

}
</script>
<style lang="less" scoped>
@import "../../plugins/timeline/timeline.css";
@import "../../plugins/timeline-item/timeline-item.css";
.el-table {
  margin: 20px 0;
}
</style>
