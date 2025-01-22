<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>
    <el-button @click="batchSave" v-if="draggable">批量保存</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :draggable="draggable"
      :allow-drop="allowDrop"
    >
      > >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            新增
          </el-button>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => edit(data)"
          >
            修改
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            删除
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="title"
      :visible.sync="dialogFormVisible"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <el-form :model="category">
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <el-form :model="category">
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
//这里可以导入其他文件（比如： 组件， 工具 js， 第三方插件 js， json文件， 图片文件等等）
//例如： import 《组件名称》 from '《组件路径》 ';
import {
  listCategoryTree,
  removeMenuByIds,
  addCategory,
  updateCategory,
  getCategory,
} from "@/api/modules/product/category";

export default {
  //import 引入的组件需要注入到对象中才能使用
  components: {},
  props: {},
  data() {
    //这里存放数据
    return {
      maxLevel: 0,
      draggable: false,
      title: "",
      dialogType: "", //edit,add
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: "",
        catId: null,
      },
      dialogFormVisible: false,
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  //计算属性 类似于 data 概念
  computed: {},
  //监控 data 中的数据变化
  watch: {},
  //方法集合
  methods: {
    allowDrop(draggingNode, dropNode, type) {
      //1 被拖动的当前节点以及所在的父节点总层数不能大于3

      //1 被拖动的当前节点总层数
      console.log("allowDrop:", draggingNode, dropNode, type);

      var level = this.countNodeLevel(draggingNode);

      // 当前正在拖动的节点+父节点所在的深度不大于3即可
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      console.log("深度:", deep);

      // this.maxLevel
      if (type == "innner") {
        // console.log(
        //   `this.maxLevel: ${this.maxLevel}; draggingNode.data.catLevel:${draggingNode.data.catLevel};dropNode.level: ${dropNode.level}`
        // );
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    countNodeLevel(node) {
      // 找到所有子节点，求出最大深度
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countNodeLevel(node.childNodes);
        }
      }
    },
    getMenus() {
      listCategoryTree().then(({ data }) => {
        this.menus = data;
      });
    },
    append(data) {
      this.dialogType = "add";
      this.title = "新增分类";
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.catId = null;
      this.category.name = null;
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
      this.dialogFormVisible = true;
    },
    addCategory() {
      addCategory(this.category).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单保存成功!",
        });
        this.dialogFormVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },

    edit(data) {
      var { catId, icon, name, productUnit, parentCid } = data;
      this.dialogType = "edit";
      this.title = "修改分类";
      //修改前根据与catId查询数据
      getCategory(catId).then(({ data }) => {
        console.log("data", data);
        this.category.catId = catId;
        this.category.icon = icon;
        this.category.name = name;
        this.category.productUnit = productUnit;
        this.category.parentCid = parentCid;
        this.dialogFormVisible = true;
      });
    },

    editCategory() {
      var { catId, name, icon, productUnit } = this.category;
      updateCategory({ catId, name, icon, productUnit }).then(({ data }) => {
        this.$message({
          type: "success",
          message: "菜单修改成功!",
        });
        this.dialogFormVisible = false;
        this.getMenus();
        this.expandedKey = [this.category.parentCid];
      });
    },

    submitData() {
      this.dialogFormVisible = true;
      if (this.dialogType == "edit") {
        console.log("修改");
        this.editCategory();
      }
      if (this.dialogType == "add") {
        console.log("新增");
        this.addCategory();
      }
    },

    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】当前菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          removeMenuByIds(ids).then(({ data }) => {
            this.$message({
              type: "success",
              message: "菜单删除成功!",
            });
          });
          this.getMenus();
          this.expandedKey = [node.parent.data.catId];
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
  //生命周期 - 创建完成（可以访问当前 this 实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（可以访问 DOM 元素）
  mounted() {},
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有 keep-alive 缓存功能， 这个函数会触发
};
</script>
<style scoped></style>
