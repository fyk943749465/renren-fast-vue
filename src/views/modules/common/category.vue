<template>
    <el-tree :data="data" :props="defaultProps" @node-click="nodeclick"></el-tree>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      data: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  computed: {},
  watch: {},
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.data = data.data
      })
    },
    nodeclick(data, node, component) {
      console.log('category click:', data, node, component)
      // send event to parent
      this.$emit('tree-node-click', data, node, component)
    }
  },
  created() {
    this.getMenus()
  },
  beforeCreate() {},
  beforeMount() {},
  beforeUpdate() {},
  updated() {},
  beforeDestory() {},
  destoryed() {},
  activated() {}
}
</script>

<style scoped>

</style>
