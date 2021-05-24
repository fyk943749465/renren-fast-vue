<template>
<el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
      data: [
        {
          label: 'Level one 1',
          children: [{
            label: 'Level two 1-1',
            children: [{
              label: 'Level three 1-1-1'
            }]
          }]
        }, {
          label: 'Level one 2',
          children: [{
            label: 'Level two 2-1',
            children: [{
              label: 'Level three 2-1-1'
            }]
          }, {
            label: 'Level two 2-2',
            children: [{
              label: 'Level three 2-2-1'
            }]
          }]
        }, {
          label: 'Level one 3',
          children: [{
            label: 'Level two 3-1',
            children: [{
              label: 'Level three 3-1-1'
            }]
          }, {
            label: 'Level two 3-2',
            children: [{
              label: 'Level three 3-2-1'
            }]
          }]
        }
      ],
      defaultProps: {
        children: 'children',
        label: 'label'
      }
    }
  },
  methods: {
    handleNodeClick(data) {
      console.log(data)
    },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(data => console.log('get mueu data.....', data))
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