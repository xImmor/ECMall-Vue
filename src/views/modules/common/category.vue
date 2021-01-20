<template>
  <el-tree
    ref="tree"
    :data="menus"
    :props="defaultProps"
    node-key="catId"
    @node-click="nodeClick">
  </el-tree>
</template>

<script>
export default {
  name: "category",
  data() {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      defaultExpandedKeys: [],
    }
  },
  created() {
    this.getMenus();
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
    },
    nodeClick(data, node, component) {
      this.$emit('tree-node-click', data, node, component)
    }
  }
}
</script>

<style scoped>

</style>
