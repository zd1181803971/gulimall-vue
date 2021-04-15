<template>
  <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
export default {
  data () {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  methods: {
    handleNodeClick (data) {
      console.log(data)
    },
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      //  {}: 将返回的数据中的data属性解构出来
      }).then(({data}) => {
        console.log(data.data)
        this.menus = data.data
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
