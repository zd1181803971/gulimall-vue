<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey">
     <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)">
            edit
          </el-button>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length === 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
<!--        <el-form-item label="分类名称">-->
<!--          <el-input v-model="category.name" autocomplete="off"></el-input>-->
<!--        </el-form-item>-->
<!--        <el-form-item label="分类名称">-->
<!--          <el-input v-model="category.name" autocomplete="off"></el-input>-->
<!--        </el-form-item>-->
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="submitData()">确 定</el-button>
  </span>
    </el-dialog>
  </div>

</template>

<script>
export default {
  data () {
    return {
      title: '',
      dialogType: '', // edit add
      category: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: ''
      },
      dialogVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
        //  {}: 将返回的数据中的data属性解构出来
      }).then(({data}) => {
        console.log(data.data)
        this.menus = data.data
      })
    },
    edit (data) {
      this.title = '修改分类'
      this.dialogType = 'edit'
      this.dialogVisible = true
      this.category.name = data.name
      this.category.catId = data.catId
    },
    append (data) {
      console.log('append', data)
      this.title = '添加分类'
      this.dialogType = 'add'
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
    },
    submitData () {
      if (this.dialogType === 'add') {
        this.addCategory()
      }
      if (this.dialogType === 'edit') {
        this.editCategory()
      }
    },
    editCategory () {
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单修改成功',
          type: 'success'
        })
        // 关闭对话框
        this.dialogVisible = false

        // 刷新出新的菜单
        this.getMenus()
        // 设置需要默认展开的菜单、
        this.expandedKey = [this.category.parentCid]
      })
    },
    // 添加三级分类数据
    addCategory () {
      console.log('提交的三级分类', this.category)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单保存成功',
          type: 'success'
        })
        // 关闭对话框
        this.dialogVisible = false

        // 刷新出新的菜单
        this.getMenus()
        // 设置需要默认展开的菜单、
        this.expandedKey = [this.category.parentCid]
      })
    },
    remove (node, data) {
      let ids = [data.catId]
      console.log('remove', node, data)
      this.$confirm(`是否删除[${data.name}]菜单？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            message: '菜单删除成功',
            type: 'success'
          })
          // 刷新出新的菜单
          this.getMenus()
          // 设置需要默认展开的菜单、
          this.expandedKey = [node.parent.data.catId]
        })
      }).catch(() => {

      })
    }
  },
  created () {
    this.getMenus()
  }
}
</script>

<style scoped>

</style>
