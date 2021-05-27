<template>
<div>
  <el-tree :data="data" :props="defaultProps" show-checkbox node-key="catId" :expand-on-click-node="false" :default-expanded-keys="expandedKey">
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
  <el-dialog :title="title" :visible.sync="dialogVisible" width="30%">
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
    append(data) {
      console.log('append', data)
      this.dialogType = 'add'
      this.title = 'Add Category'
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
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