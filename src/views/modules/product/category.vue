<template>
  <div>
    <el-tree :data="menus" 
    :props="defaultProps" 
    :expand-on-click-node="false" 
    show-checkbox node-key="catId"
    :default-expanded-keys="expandedKey"
    draggable
    :allow-drop = "allowDrop"
    @node-drop="handleDrop"
    v-loading="categoryLoading">
      <span class="custom-tree-node" slot-scope="{ node, data }">
          <span>{{ node.label }}</span>
          <span>
            <el-button
              v-if="node.level <= 2"
              type="text"
              size="mini"
              @click="() => append(data)">
              Append
            </el-button>
            <el-button
              type="text"
              size="mini"
              @click="edit(data)">
              edit
            </el-button>
            <el-button
              v-if="node.childNodes.length == 0"
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
      width="30%"
      :close-on-click-modal="false">
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
  components: {},
  props: {},
  data() {
    return {
      categoryLoading: true,
      updateNodes: [],
      maxLevel: 0,
      title:"",
      dialogType: "", // edit, add
      category: {
        catId: "",
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        icon:"",
        productUnit: "",
        sort: 0
      },
      dialogVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    };
  },
  methods: {
      handleNodeClick(data) {
          console.log(data);
      },
      allowDrop(draggingNode, dropNode, type) {
        //1、被拖动的当前节点以及所在的父节点总层数不能大于san
        // 被拖动的当前姐点总层数
        console.log("allowDrop", draggingNode, dropNode, type);
        this.countNodeLevel(draggingNode.data);
        // 当前拖动的节点+父节点所在的深度不大于3就可以
        let deep = (this.maxLevel - draggingNode.data.catLevel) + 1
        console.log("deep", deep);
        console.log("this.maxLevel", this.maxLevel);
        if (type == 'inner') {
          return (deep + dropNode.level) <= 3;
        } else {
          return (deep + dropNode.parent.level) <= 3;
        }
      },
      countNodeLevel(node) {
        if(node.children != null && node.children.length > 0) {
          for(let i = 0; i < node.children.length; i++) {
            if(node.children[i].catLevel > this.maxLevel) {
              this.maxLevel = node.children[i].catLevel;
            }
            this.countNodeLevel(node.children[i])
          }
        }
      },
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
      append(data) {
        console.log("append", data);
        this.dialogType = "add";
        this.title = "添加分类";
        this.dialogVisible = true;
        this.category.parentCid = data.catId;
        this.category.catLevel = data.catLevel * 1 + 1;
        this.category.catId = null;
        this.category.name = "";
        this.category.icon = "";
        this.category.productUnit = "";
        this.category.sort = 0;
        this.category.showStatus = 1;
      },
      edit(data) {
        console.log("要修改的数据", data)
        this.dialogType = "edit";
        this.title = "修改分类"
        this.dialogVisible = true;

        this.$http({
          url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
          method: 'get'
        }).then(({data}) => {
          console.log("要回显的数据", data);
          this.category.name = data.data.name;
          this.category.icon = data.data.icon;
          this.category.productUnit = data.data.productUnit;
          this.category.parentCid = data.data.parentCid;
          this.category.catId = data.data.catId;
        })
      },
      submitData() {
        if (this.dialogType == "add") {
          this.addCategory();
        }
        if (this.dialogType == "edit") {
          this.editCategory();
        }
      },
      // 添加三级分类的方法
      addCategory() {
        console.log("提交的三级分类数据", this.category);
         this.$http({
              url: this.$http.adornUrl('/product/category/save'),
              method: 'post',
              data: this.$http.adornData(this.category, false)
          }).then(({data}) => {
              this.$message({
                message: "菜单保存成功",
                type: "success"
              })
              this.dialogVisible = false;
              this.getMenus();
               // 设置需要默认展开的菜单
              this.expandedKey = [this.category.parentCid]
          })
      },
      // 修改三级分类的方法
      editCategory() {
        console.log("提交的三级分类数据", this.category);
        var {catId, name, icon, productUnit} = this.category; 
        var data = {
          catId: catId,
          name: name,
          icon: icon,
          productUnit: productUnit
        }
        this.$http({
            url: this.$http.adornUrl('/product/category/update'),
            method: 'post',
            data: this.$http.adornData(data, false)
        }).then(({data}) => {
            this.$message({
              message: "菜单修改成功",
              type: "success"
            })
            this.dialogVisible = false;
            this.getMenus();
              // 设置需要默认展开的菜单
            this.expandedKey = [this.category.parentCid]
        })
      },
      remove(node, data) {
        var ids = [data.catId]

         this.$confirm(`是否删除【${data.name}】菜单?`, '提示', {
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
              type: 'success',
              message: '删除成功!'
            });
            console.log("删除成功");
            this.getMenus();
            // 设置需要默认展开的菜单
            this.expandedKey = [node.parent.data.catId]
          })
        }).catch(() => {

        });
      },
      handleDrop(draggingNode, dropNode, dropType, ev) {
        console.log('tree drop: ', dropNode.label, dropType);
        // 1、当前节点最新的父节点id
        let pCid = 0;
        let siblings = null;
        if (dropType == 'before' || dropType == 'after') {
          pCid = dropNode.parent.data.catId == undefined ? 0 : dropNode.parent.data.catId
           siblings = dropNode.parent.childNodes
        } else {
          pCid = dropNode.data.catId;
          siblings = dropNode.childNodes
        }
        // 2、当前拖拽节点的最新顺序
        for (let i = 0; i < siblings.length; i++) {
          if (siblings[i].data.catId == draggingNode.data.catId) {
            let catLevel = draggingNode.level;
            if (siblings[i].level != draggingNode.level) { 
              // 当前节点的层级发生变化
              catLevel = siblings[i].level;
              // 修改子节点的层级
              this.updateChildNodeLevel(siblings[i]);
            }
            this.updateNodes.push({catId: siblings[i].data.catId, sort:i, parentCid: pCid, catLevel: catLevel})
          } else {
            this.updateNodes.push({catId: siblings[i].data.catId, sort:i})
          }
          
        }
        // 3、当前拖拽节点的最新层级
        console.log("updateNodes", this.updateNodes)
        this.$http({
          url: this.$http.adornUrl('/product/category/update/sort'),
          method: "post",
          data: this.$http.adornData(this.updateNodes, false)
        }).then(({data}) => {
            this.$message({
              type: 'success',
              message: '菜单顺序修改成功!'
            });
            // 刷新菜单
            this.getMenus();
            // 设置需要默认展开的菜单
            this.expandedKey = [pCid]
        });
      },
      updateChildNodeLevel(node) {
        if (node.childNodes.length > 0){
          for (let i = 0; i < node.childNodes.length; i++) {
            var cNode = node.childNodes[i].data;
            this.updateNodes.push({catId:cNode.catId, catLevel:node.childNodes[i].level})
            this.updateChildNodeLevel(node.childNodes[i]);        
          }
        }
        
    }
  },
  watch: {},
  computed: {},
  created() {
    console.log("111111111111111111111111111");
      this.getMenus();
  },
  mounted() {}
  
}
</script>
<style lang="scss" scoped>
</style>