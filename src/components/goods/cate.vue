<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加分类</el-button>
        </el-col>
      </el-row> 
      <!-- 表格区域 -->
      <tree-table class="treeTable" :data="cateList" :columns="columns" :selection-type="false" :expand-type="false" 
      show-index index-text="#" border :show-row-hover="false">
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color:green"></i>
          <i class="el-icon-error" v-else style="color:red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag type="primary" size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[1, 3, 5, 10]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total= total>
    </el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //商品分类的数据列表
      cateList: [],
      //查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      //总数据条数
      total: 0,
      //为table指定列的定义
      columns: [{
          label: '分类名称',
          prop: 'cat_name'
      },
        {
          label: '是否有效',
          type: 'template',//将当前列定义为模板列
          template: 'isok',//模板名称
        },
        {
          label: '排序',
          type: 'template',//将当前列定义为模板列
          template: 'order',//模板名称
        },
        {
          label: '操作',
          type: 'template',//将当前列定义为模板列
          template: 'opt',//模板名称
        }
      ]
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) this.$message.error('获取商品分类失败')
      this.cateList = res.data.result
      this.total = res.data.total
    },
    //监听pagesize改变的事件
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    //监听pagenum的改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    }
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px
}
</style> 

