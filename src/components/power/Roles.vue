<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>权限管理</el-breadcrumb-item>
        <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 用户角色列表区域 -->
      <el-table :data="roleList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row :class= "['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id" @close="removeRightById(scope.row, item1.id)">
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable>{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="9">
                <!-- 通过for循环嵌套渲染二级权限 -->
                <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id" @close="removeRightById(scope.row, item2.id)">
                  <el-col :span="6">
                    <el-tag closable type="success">{{ item2.authName }}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag closable @close="removeRightById(scope.row, item3.id)" type="warning" v-for="(item3, i3) in item2.children" :key="item3.id">{{ item3.authName }}</el-tag>

                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加角色的对话框 -->
    <el-dialog title="添加角色" width="50%" >
    </el-dialog>
    <!-- 分配权限的对话框 -->
    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
      <!-- 树形控件 -->
      <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary"  @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //所有角色列表数据
      roleList: [],
      //控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      //所有权限数据
      rightsList: [],
      //树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      //默认选中的节点id值
      defKeys: [],
      //当前即将分配权限角色id
      roleId: ''
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    //获取所有角色列表
    async getRoleList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败！')
      this.roleList = res.data
    },
    async removeRightById(role, rightId) {
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)

      if (confirmResult !== 'confirm') {
        return this.$message.info('用户取消了删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      role.children = res.data
    },
    async showSetRightDialog(role) {
      //获取所有权限的数据
      this.roleId = role.id
      const { data: res } = await this.$http.get('rights/tree')
      if(res.meta.status !== 200) return this.$message.error('权限列表请求失败')
      this.rightsList = res.data
      this.getDefKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    //通过递归的形式，获取角色三级权限
    getDefKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      node.children.forEach(item => this.getDefKeys(item, arr))
    },
    //监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    //点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedNodes()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles / ${this.roleId}/rights`, idStr)
      if (res.meta.status !== 200) return this.$message.error('分配权限失败')
      this.$message.success('更新权限成功')
      this.getRoleList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
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
