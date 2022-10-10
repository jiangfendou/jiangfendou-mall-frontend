<template>
  <el-tree :data="menus" 
    :props="defaultProps" 
    node-key="catId"
    ref="menuTree"
    @node-click="nodeClick"
    v-loading="categoryLoading">
  </el-tree>
</template>

<script>


export default {

  components: {},
  
  data() {
    //这里存放数据
    return {
      categoryLoading: true,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    };
  },
  watch:{
    
  },
  //方法集合
  methods: {
    getMenus() {
      this.$http({
          url: this.$http.adornUrl('/product/category/list/tree'),
          method: 'get',
      }).then(({data}) => {
          console.log("成功获取菜单数据，，，", data);
          this.menus = data.data;
          this.categoryLoading = false
      })
    },
    nodeClick(data, node, component) {
      console.log("点击事件");
      console.log(data, node, component);
      // 像父组件发送事件
      this.$emit("treeNodeClick", data, node, component)
    }
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  }
};
</script>
<style scoped>
</style>