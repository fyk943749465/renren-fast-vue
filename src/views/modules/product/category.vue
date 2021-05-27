<template>
<div>
  <el-tree :data="data"
    :props="defaultProps"
    show-checkbox node-key="catId"
    :expand-on-click-node="false"
    draggable
    :allow-drop="allowDrop"
    :default-expanded-keys="expandedKey"
    @node-drop="handleDrop"
  >
    <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="data.childCategoryList.length != 0"
            type="text"
            size="mini"
            @click="() => append(data)">
            Append
          </el-button>
          <el-button v-else
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(node, data)">
            Edit
          </el-button>
        </span>
      </span>
  </el-tree>
  <el-dialog :title="title" :visible.sync="dialogVisible" width="30%" :close-on-click-modal="false">
    <el-form :model="category">
      <el-form-item label="Category Name">
        <el-input v-model="category.name" auto-complete="off"></el-input>
      </el-form-item>
      <el-form-item label="Icon">
        <el-input v-model="category.icon" auto-complete="off"></el-input>
      </el-form-item>
      <el-form-item label="Product Unit">
        <el-input v-model="category.productUnit" auto-complete="off"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="dialogVisible = false">Cancel</el-button>
      <el-button type="primary" @click="submitData">Confirm</el-button>
    </span>
  </el-dialog>
</div>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      updateNodes: [],
      maxLevel: 0,
      title: '',
      dialogType: '', // edit, add
      category: {name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, icon: '', productUnit: '', catId: null},
      dialogVisible: false,
      data: [],
      expandedKey: [],
      defaultProps: {
        children: 'childCategoryList',
        label: 'name'
      }
    }
  },
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.data = data.data
      })
    },
    allowDrop(draggingNode, dropNode, type) {
      this.maxLevel = this.countDraggingNodeLevel(draggingNode.data)
      let deep = this.maxLevel - draggingNode.data.catLevel + 1
      console.log(type)
      if (type === 'inner') {
        return dropNode.data.catLevel + deep <= 3
      } else {
        return dropNode.data.catLevel + deep - 1 <= 3
      }
    },
    countDraggingNodeLevel(data) {
      if (data.childCategoryList !== null && data.childCategoryList.length !== 0) {
        for (let i = 0; i < data.childCategoryList.length; ++i) {
          let currentLevel = this.countDraggingNodeLevel(data.childCategoryList[i])
          if (this.maxLevel < currentLevel) {
            this.maxLevel = currentLevel
          }
        }
        return this.maxLevel
      }
      return data.catLevel
    },
    handleDrop(draggingNode, dropNode, dropType, ev) {
      // 1. find the current node's new parent node and siblings
      console.log('draggingNode', draggingNode)
      let pCid = 0
      let siblings = null
      if (dropType === 'before' || dropType === 'after') {
        pCid = dropNode.parent.data.catId === undefined ? 0 : dropNode.parent.data.catId
        siblings = dropNode.parent.childNodes
      } else {
        pCid = dropNode.data.catId
        siblings = dropNode.childNodes
      }
      // 2. find the current node's new order and update catLevel
      for (let i = 0; i < siblings.length; ++i) {
        if (siblings[i].data.catId === draggingNode.data.catId) {
          let catLevel = draggingNode.level
          if (siblings[i].level !== draggingNode.level) {
            catLevel = siblings[i].level
            this.updateChildNodeLevel(siblings[i])
          }
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i, parentCid: pCid, catLevel: catLevel})
        } else {
          this.updateNodes.push({catId: siblings[i].data.catId, sort: i})
        }
      }
      console.log('updateNodes', this.updateNodes)
      // 3. update database
      this.$http({
        url: this.$http.adornUrl('/product/category/update/sort'),
        method: 'post',
        data: this.$http.adornData(this.updateNodes, false)
      }).then(({data}) => {
        this.$message({
          message: 'sort modify success',
          type: 'success'
        })
        this.getMenus()
        this.expandedKey = [pCid]
      })
    },
    updateChildNodeLevel(node) {
      if (node.childNodes !== null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; ++i) {
          var cNode = node.childNodes[i].data
          this.updateNodes.push({catId: cNode.catId, catLevel: node.childNodes[i].level})
          this.updateChildNodeLevel(node.childNodes[i])
        }
      }
    },
    append(data) {
      console.log('append', data)
      this.dialogType = 'add'
      this.title = 'Add Category'
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      this.category.catId = null
      this.category.name = ''
      this.category.productUnit = ''
      this.category.icon = ''
      this.category.sort = 0
      this.category.showStatus = 1
    },
    submitData() {
      if (this.dialogType === 'add') {
        this.addCategory()
      }
      if (this.dialogType === 'edit') {
        this.editCategory()
      }
    },
    addCategory() {
      console.log('add data:', this.category)
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          type: 'success',
          message: 'Save success'
        })
        this.dialogVisible = false
        this.getMenus()
        this.expandedKey = [this.category.parentCid]
      })
    },
    editCategory() {
      var {catId, name, icon, productUnit} = this.category
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({catId, name, icon, productUnit}, false)
      }).then(({data}) => {
        this.$message({
          message: 'menu edit success',
          type: 'success'
        })
        this.dialogVisible = false
        this.getMenus()
        this.expandedKey = [this.category.parentCid]
      })
    },
    remove(node, data) {
      var ids = [data.catId]
      this.$confirm(`This ${data.name} will permanently delete the file. Continue?`, 'Warning', {
        confirmButtonText: 'OK',
        cancelButtonText: 'Cancel',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            type: 'success',
            message: 'Delete completed'
          })
          this.getMenus()
          this.expandedKey = [node.parent.data.catId]
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: 'Delete canceled'
        })
      })
    },
    edit(node, data) {
      this.title = 'Edit Category'
      this.dialogType = 'edit'
      this.dialogVisible = true
      console.log('edit', node, data)
      // you must send request and get the back data,
      // because not only one use the system and the data has been modified, so the data we see now it's fault
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({ data }) => {
        console.log('servcer back data', data)
        this.category.name = data.category.name
        this.category.catId = data.category.catId
        this.category.icon = data.category.icon
        this.category.productUnit = data.category.productUnit
        this.category.parentCid = data.category.parentCid
      })
      // this.category.name = data.name
      // this.category.catId = data.catId
    }
  },
  computed: {},
  watch: {},
  created() {
    this.getMenus()
  },
  mounted() {

  },
  beforeCreate() {

  },
  beforeMount() {

  },
  beforeUpdate() {

  },
  updated() {

  },
  beforeDestroy() {

  },
  destroyed() {

  },
  activated() { // if the page has keep-alive cache function, this function will be called

  }
}
</script>

<style>

</style>