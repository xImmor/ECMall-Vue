<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽">
    </el-switch>
    <el-button
      v-if="draggable"
      @click="batchSave">
      批量保存
    </el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      node-key="catId"
      show-checkbox
      :draggable="draggable"
      @node-drop="handleDrop"
      :allow-drop="allowDrop"
      :default-expanded-keys="defaultExpandedKeys"
      :expand-on-click-node="false">
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level !== 3"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)">
            Edit
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
      :close-on-click-modal="false"
      :title="dialogTitle"
      :visible.sync="dialogVisible"
      width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="submitData">确 定</el-button>
  </span>
    </el-dialog>
  </div>

</template>

<script>
export default {
  name: 'category',
  created() {
    this.getMenus()
  },
  data() {
    return {
      pCid: [],
      draggable: false,
      updateNodes: [],
      maxLevel: 0,
      dialogTitle: "",
      dialogType: "",
      dialogVisible: false,
      category: {
        catId: null,
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        productUnit: "",
        icon: "",
      },
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      defaultExpandedKeys: [],
    }
  },
  methods: {
    batchSave() {
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({data}) => {
        this.$message({
          message: `菜单顺序等新修改成功`,
          type: 'success'
        });
        //刷新菜单
        this.getMenus();
        this.defaultExpandedKeys = this.pCid;
        this.updateNodes = [];
        this.maxLevel = 0;
        // this.pCid = 0;
      })
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log('tree drop: ', draggingNode, dropNode, dropType);
      let pCId = 0;
      let sibling = null;
      if (dropType === "before" || dropType === "after") {
        pCId = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId;
        sibling = dropNode.parent.childNodes;
      } else {
        pCId = dropNode.data.catId;
        sibling = dropNode.childNodes;
      }
      this.pCid.push(pCId);
      for (let i = 0; i < sibling.length; i++) {
        if (sibling[i].data.catId === draggingNode.data.catId) {
          let catLevel = draggingNode.level;
          if (sibling[i].level !== draggingNode.level) {
            catLevel = sibling[i].level;
            this.updateChildNodeLevel(sibling[i]);
          }
          this.updateNodes.push({catId: sibling[i].data.catId, sort: i, parentCid: pCId, catLevel: catLevel});
        } else {
          this.updateNodes.push({catId: sibling[i].data.catId, sort: i});
        }
      }
      console.log(this.updateNodes)
    },
    updateChildNodeLevel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          let cNode = node.childNodes[i].data;
          this.updateNodes.push({catId: cNode.catId, catLevel: node.childNodes[i].level});
          this.updateChildNodeLevel(node.childNodes[i]);
        }
      }
    },
    allowDrop(draggingNode, dropNode, type) {
      this.countNodeLevel(draggingNode);
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      if (type === "inner") {
        return (deep + dropNode.level) <= 3;
      }
      return (deep + dropNode.parent.level) <= 3;
    },
    countNodeLevel(node) {
      if (node.childNodes && node.childNodes.length !== 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countNodeLevel(node.childNodes[i]);
        }
      }
    },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
    },
    submitData() {
      if (this.dialogType === "add") {
        this.addCategory();
      }
      if (this.dialogType === "edit") {
        this.editCategory();
      }
    },
    append(data) {
      this.dialogVisible = true;
      this.dialogType = "add";
      this.dialogTitle = "添加分类";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel + 1;
      this.category.name = "";
      this.category.catId = null;
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
    },
    addCategory() {
      console.log(this.category)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          message: `${this.category.name}菜单新增成功`,
          type: 'success'
        });
        this.dialogVisible = false;
        this.getMenus();
        this.defaultExpandedKeys = [this.category.parentCid];
      })
    },
    editCategory() {
      let {catId, name, icon, productUnit} = this.category;
      let data = {catId, name, icon, productUnit};
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData(data, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单修改成功',
          type: 'success'
        });
        this.dialogVisible = false;
        this.getMenus();
        this.defaultExpandedKeys = [this.category.parentCid];
      })
    },
    edit(data) {
      console.log(data);
      this.dialogVisible = true;
      this.dialogType = "edit";
      this.dialogTitle = "修改分类";
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({data}) => {
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
        this.category.catLevel = data.data.catLevel;
        this.category.sort = data.data.sort;
        this.category.showStatus = data.data.showStatus;
      })
    },
    remove(node, data) {
      this.$confirm(`是否删除【${data.name}】菜单?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let ids = [data.catId];
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            message: '菜单删除成功',
            type: 'success'
          });
          this.getMenus();
          this.defaultExpandedKeys = [node.parent.data.catId];
        })
      }).catch(() => {

      })
      console.log(node, "-", data)
    },
  }
}
</script>

<style scoped>

</style>
