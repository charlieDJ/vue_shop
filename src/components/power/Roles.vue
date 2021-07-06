<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddRoleDialog">
            添加角色
          </el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table :data="roleList" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag>{{ item1.authName }}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级、三级权限 -->
              <el-col :span="19">
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag type="success">{{ item2.authName }}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showRightEditDialog(scope.row.roleId)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="deleteRoleById(scope.row.roleId)"
              >删除</el-button
            >
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="roleAddDialogVisible"
      width="30%"
      @close="roleAddDialogClosed"
      append-to-body
    >
      <el-form :model="roleAddForm" ref="roleAddFormRef" label-width="70px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="roleAddForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="roleAddForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="roleAddDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改角色对话框 -->
    <el-dialog
      title="修改角色"
      :visible.sync="roleEditDialogVisible"
      width="50%"
      @close="roleEditDialogClosed"
    >
      <el-form :model="roleEditForm" ref="roleEditFormRef" label-width="70px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="roleEditForm.roleName" disabled></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="roleEditForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="roleEditDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog title="提示" :visible.sync="showRightDialogVisible" width="50%"
    @close="resetRightCheckedFileds">
      <el-tree
        :data="rightsList"
        :props="treeProps" show-checkbox node-key="id" default-expand-all
        :default-checked-keys="checkedKeys" ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="showRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRights"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 角色列表
      roleList: [],
      roleEditForm: {},
      roleEditDialogVisible: false,
      roleAddDialogVisible: false,
      roleAddForm: {
        roleName: '',
        roleDesc: ''
      },
      showRightDialogVisible: false,
      // 权限
      rightsList: [],
      // 权限树形属性
      treeProps: {
        children: 'children',
        label: 'authName'
      },
      checkedKeys: [],
      // 即将分配权限的roleID
      assignRoleId: ''
    }
  },
  created() {
    this.getRoleList()
  },
  methods: {
    async getRoleList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.roleList = res.data
      //   console.log(this.roleList)
    },
    // 显示修改角色对话框
    async showRightEditDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色信息失败')
      }
      this.roleEditForm = res.data
      this.roleEditDialogVisible = true
    },
    roleEditDialogClosed() {
      this.$refs.roleEditFormRef.resetFields()
    },
    roleAddDialogClosed() {
      this.$refs.roleAddFormRef.resetFields()
    },
    // 显示新增角色对话框
    showAddRoleDialog() {
      this.roleAddDialogVisible = true
    },
    // 修改角色
    async editRoleInfo() {
      const { data: res } = await this.$http.put(
        'roles/' + this.roleEditForm.roleId,
        {
          roleId: this.roleEditForm.roleId,
          roleName: this.roleEditForm.roleName,
          roleDesc: this.roleEditForm.roleDesc
        }
      )
      if (parseInt(res.meta.status) !== 200) {
        this.$message.error('修改角色失败')
      }
      // 隐藏对话框
      this.roleEditDialogVisible = false
      // 重新刷新用户
      this.getRoleList()
    },
    // 删除角色，缺少校验
    async deleteRoleById(id) {
      const confirmResult = await this.$confirm('是否删除角色', '删除提示', {
        confirmButtonText: '确认删除',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已经取消删除')
      }
      // 确定：confirm，取消：cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消操作')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        this.$message.error('删除角色失败')
      }
      this.$message.success('删除角色成功')
      this.getRoleList()
    },
    // 新增角色，缺少校验
    async addRoleInfo() {
      const { data: res } = await this.$http.post('roles', this.roleAddForm)
      if (parseInt(res.meta.status) !== 201) {
        this.$message.error('添加角色失败')
      }
      // 隐藏对话框
      this.roleAddDialogVisible = false
      // 重新刷新角色
      this.getRoleList()
    },
    async removeRightById(role, id) {
      // 弹框提示用户
      const confirmResult = await this.$confirm(
        '此操作将永久删除该文件, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除')
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.roleId}/rights/${id}`
      )
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败')
      }
      this.$message.success('删除成功')
      //   this.getRoleList() 此方法会关闭当前菜单
      role.children = res.data.children
    },
    // 显示分配权限对话框
    async showRightDialog(role) {
      this.assignRoleId = role.roleId
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('未获取到权限列表')
      }
      // 获取到的权限数据
      this.rightsList = res.data
      // 递归获取叶子节点
      this.getLeafKeys(role, this.checkedKeys)
      this.showRightDialogVisible = true
    },
    getLeafKeys(node, arr) {
      if (!node.children) {
        return arr.push(node.id)
      }
      // Each 大写，好坑
      node.children.forEach(item => {
        this.getLeafKeys(item, arr)
      })
    },
    // 监听权限分配框关闭事件
    resetRightCheckedFileds() {
      this.checkedKeys = []
    },
    // 为角色分配权限
    async saveRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      console.log(this.assignRoleId)
      const { data: res } = await this.$http.post(`roles/${this.assignRoleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) { return this.$message.error('分配权限失败') }

      this.$message.success('分配权限成功')
      this.getRoleList()
      // 关闭对话框
      this.showRightDialogVisible = false
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
