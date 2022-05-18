<template>
  <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
export default {
  components: {},
  props: {},
  data() {
    return {
        data: [{
          label: '一级 1',
          children: [{
            label: '二级 1-1',
            children: [{
              label: '三级 1-1-1'
            }]
          }]
        }, {
          label: '一级 2',
          children: [{
            label: '二级 2-1',
            children: [{
              label: '三级 2-1-1'
            }]
          }, {
            label: '二级 2-2',
            children: [{
              label: '三级 2-2-1'
            }]
          }]
        }, {
          label: '一级 3',
          children: [{
            label: '二级 3-1',
            children: [{
              label: '三级 3-1-1'
            }]
          }, {
            label: '二级 3-2',
            children: [{
              label: '三级 3-2-1'
            }]
          }]
        }],
        defaultProps: {
          children: 'children',
          label: 'label'
        }
      };
    },
    methods: {
        handleNodeClick(data) {
            console.log(data);
        },
        getMenus() {
            this.$http({
                url: this.$http.adornUrl('/product/category/list/tree'),
                method: 'get',
            }).then(({data}) => {
                console.log("成功获取菜单数据，，，", data);
            })
        }
    },
    watch: {},
    computed: {},
    created() {
        this.getMenus();
    },
    mounted() {}
};
</script>
<style lang="scss" scoped>
</style>