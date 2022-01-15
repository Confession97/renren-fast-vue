<!--  -->
<template>
  <div>
    <el-tree
      :data="menus"
      show-checkbox
      node-key="catId"
      :props="defaultProps"
      :expand-on-click-node="false"
      :default-expanded-keys="expandKey"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog title="提示" :visible.sync="dialogFormVisible">
    <el-form :model="category">
        <el-form-item label="分类名称" >
            <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
    </el-form>

    <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategory">确 定</el-button>
    </div>
    </el-dialog>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    //这里存放数据
    return {
      category:{
          name:"",
          parentCid:0,
          catLevel:0,
          showStatus:1,
          sort:0
      },
      menus: [],
      expandKey:[],
      dialogFormVisible:false,
      defaultProps: {
        children: "children",
        label: "name",
      }
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log(data);
        this.menus = data.tree;
      });
    },
    addCategory(){
        console.log("提交的三级分类数据",this.category)
        this.$http({
            url : this.$http.adornUrl("/product/category/save"),
            method: "post",
            data : this.$http.adornData(this.category,false)
        }).then(({data}) => {
            this.$message({
                type: 'success',
                message:'菜单保存成功'
            })
            this.dialogFormVisible = false
            this.expandKey = [this.category.parentCid]
            this.getMenus()
        })
    },
    //data是点击的数据
    append(data) {
        console.log("append",data)
        this.dialogFormVisible = true
        this.category.parentCid = data.catId
        this.category.catLevel = data.catLevel * 1 + 1
    },
    remove(node, data) {
      console.log(node);
      console.log(data);

      this.$confirm("确认是否删除此产品项吗?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          var ids = [data.catId];
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            console.log("删除成功");
            this.$message({
                type: "success",
                message: "删除成功!",
            });
        this.expandKey=[node.parent.data.catId]
        this.getMenus();

        });
          
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });

    }
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
.custom-tree-node {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: space-between;
  font-size: 14px;
  padding-right: 8px;
}
</style>