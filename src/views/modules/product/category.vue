<template>
  <div>
    <el-switch
      v-model="enableDraggle"
      active-text="开启拖拽功能"
      inactive-text="关闭拖拽功能"
    ></el-switch>
    <el-button v-if="enableDraggle" @click="saveDraggle"
      >保存拖拽结果</el-button
    >
    <el-button type="danger" @click="deleteCategory">批量删除</el-button>
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      :draggable="enableDraggle"
      :allow-drop="allowDrop"
      @node-drop="handleDrop"
      ref="menuTree"
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

          <el-button type="text" size="mini" @click="() => update(data)">
            Edit
          </el-button>

          <el-button
            type="text"
            size="mini"
            @click="() => testCountLevel(data)"
          >
            Count Level
          </el-button>
        </span>
      </span>
    </el-tree>

    <!-- 点击append之后显示(弹出)的对话框 -->
    <el-dialog title="添加分类" :visible.sync="appendDialogVisible" width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="appendDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategoryMethod">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 点击Edit之后显示(弹出)的对话框 -->
    <el-dialog
      title="编辑分类"
      :close-on-click-modal="false"
      :visible.sync="updateDialogVisible"
      width="30%"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off" />
        </el-form-item>
        <el-form-item label="分类图标">
          <el-input v-model="category.icon" autocomplete="off" />
        </el-form-item>
        <el-form-item label="分类计量单位">
          <el-input v-model="category.productUnit" autocomplete="off" />
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="updateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="updateCategoryMethod"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  componets: {},
  props: {},
  data() {
    return {
      menus: [],
      appendDialogVisible: false, // 控制添加对话框的显示与否
      updateDialogVisible: false, // 控制修改对话框的显示与否
      updateNodes: [],
      enableDraggle: false,
      pCid: [], // 当前节点最新的父节点id
      category: {
        catId: null,
        name: "",
        parentCid: 0, // 父节点id
        catLevel: 0, // 等级
        showStatus: 1, //逻辑删除显示
        sort: 0,
        icon: "", // 图标
        productUnit: "", // 计量单位
        productCount: 0, // 数量
      },
      expandedKey: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },

  computed: {},

  watch: {},

  methods: {
    getMenus() {
      let that = this;
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        that.menus = data.data;
        // console.log(that.menus);
      });
    },

    append(data) {
      this.appendDialogVisible = true; // 显示对话框
      Object.assign(this.$data.category, this.$options.data().category);
      console.log(this.category); // 将对象还原为初始时的样子
      this.category.parentCid = data.catId; // 指定新加的分类的父节点为当前点击的节点
      this.category.catLevel = data.catLevel * 1 + 1; // 指定层级，为当前的层级 + 1, *1 是为了将字符串转换为数字类型
    },

    remove(node, data) {
      let that = this;
      // console.log("node：" + node);
      this.$confirm("确认删除该分类: " + data.name + "?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          let ids = [data.catId];

          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "delete",
            data: this.$http.adornData(ids, false),
          }).then((res) => {
            // console.log("ids = " + ids);
            // console.log("delete successfully");
            // console.log(res);
            that.getMenus(); // 更新删除操作后的数据
          });

          this.$message({
            // 删除成功弹窗
            type: "success",
            message: "删除分类成功: " + data.name,
          });

          this.expandedKey = [node.parent.data.catId]; // 删除后依然展开删去节点的父节点
        })
        .catch(() => {
          // 取消删除操作

          this.$message({
            // 弹窗
            type: "info",
            message: "已取消删除",
          });
        });
    },

    addCategoryMethod() {
      // 添加分类方法
      let that = this;
      console.log(that.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(that.category, false),
      }).then((res) => {
        console.log(res.data.code);
        console.log(res.data.msg);

        if (res.data.code === 0) {
          this.$message({
            showClose: true,
            message: "分类" + this.category.name + "添加成功",
            type: "success",
          });
        } else {
          this.$message({
            showClose: true,
            message: "添加失败! " + res.data.msg,
            type: "error",
          });
        }

        that.appendDialogVisible = false; // 关闭对话框
        this.getMenus(); // 更新菜单列表
        this.expandedKey = [this.category.parentCid]; // 将添加的分类的父节点设为默认展开
      });
    },

    update(data) {
      console.log(data);
      let that = this;
      // 从数据库中查询最新的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then((res) => {
        let temp = res.data.data;
        that.category.name = temp.name;
        that.category.catId = temp.catId;
        that.category.icon = temp.icon;
        that.category.productUnit = temp.productUnit;
        that.category.parentCid = temp.parentCid;
      });
      this.updateDialogVisible = true; // 弹出修改对话框
    },

    updateCategoryMethod() {
      let that = this;
      let { catId, name, icon, productUnit } = this.category; // 按部分属性解构

      console.log(this.category);

      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "put",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      }).then((res) => {
        console.log("The response code: " + res.data.code);
        if (res.data.code === 0) {
          this.$message({
            showClose: true,
            message: "修改成功",
            type: "success",
          });
        } else {
          this.$message({
            showClose: true,
            message: "修改失败! " + res.data.msg,
            type: "error",
          });
        }
      });

      that.updateDialogVisible = false; // 关闭对话框
      this.getMenus(); // 更新菜单列表
      this.expandedKey = [this.category.parentCid]; // 将添加的分类的父节点设为默认展开
    },

    allowDrop(
      draggingNode /* 当前节点 */,
      dropNode /* 拖拽到了哪个节点的前面 */,
      type /* 放置位置 */
    ) {
      // console.log(draggingNode, dropNode, type);

      // 1.被拖动的当前节点以及所在的父节点总层数不能大于3
      let depth =
        Math.abs(this.countNodeLevel(draggingNode) - draggingNode.level) + 1;

      if (type === "inner") {
        return depth + dropNode.level <= 3;
      } else {
        return depth + dropNode.parent.level <= 3;
      }
    },

    testCountLevel(data) {
      console.log(
        "The level of the current node is :" + this.countNodeLevel(data)
      );
    },

    countNodeLevel(node) {
      // console.log(node);
      // 递归找出所有子节点，计算最大深度
      if (!node.childNodes || !node.childNodes.length) return node.level;
      let max = 0;
      for (let children of node.childNodes) {
        max = Math.max(max, this.countNodeLevel(children));
      }
      return max;
    },

    handleDrop(draggingNode, dropNode, dropType, ev) {
      let pCid = 0;
      let neigbours = null; // 兄弟节点
      if (["after", "before"].indexOf(dropType) !== -1) {
        // 前后关系拖拽进入
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        neigbours = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        neigbours = dropNode.childNodes;
      }
      this.pCid.push(pCid);
      // 当前节点的最新顺序
      for (let i = 0; i < neigbours.length; i++) {
        if (neigbours[i].data.catId === draggingNode.data.catId) {
          //如果拖拽的是正在拖拽的节点
          let catLevel = draggingNode.level;
          if (neigbours[i].level !== draggingNode.level) {
            // 当前节点的层级发生变化
            catLevel = neigbours[i].level;
            // 修改他子节点的层级
            this.updateChildrenNodeLevel(neigbours[i]);
          }
          this.updateNodes.push({
            catId: neigbours[i].data.catId,
            sort: i,
            parentCid: this.pCid[this.pCid.length - 1],
            catLevel: catLevel,
          });
        }
        this.updateNodes.push({
          catId: neigbours[i].data.catId,
          sort: i,
        });
      }
      // 当前节点的最新层级
    },

    updateChildrenNodeLevel(node) {
      // 修改子节点的层级
      let childrens = node.childNodes;
      for (let i = 0; i < childrens; i++) {
        let cnode = childrens[i].data;
        this.updateNodes.push({
          catId: cnode.catId,
          catLevel: childrens[i].level,
        });
        this.updateChildrenNodeLevel(childrens[i]);
      }
    },

    saveDraggle() {
      let that = this;
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "put",
        data: this.$http.adornData(this.updateNodes, false),
      }).then((res) => {
        if (res.data.code === 0) {
          this.$message({
            showClose: true,
            message: "菜单修改成功",
            type: "success",
          });
          // 刷新菜单
          this.getMenus();
          this.expandedKey = that.pCid;
          this.updateNodes = [];
        } else {
          this.$message({
            showClose: true,
            message: "菜单修改失败! " + res.data.msg,
            type: "error",
          });
        }
      });
    },

    deleteCategory() {
      let checkNodes = this.$refs.menuTree.getCheckedNodes(); // 获取已选区的节点(数组)
      // console.log("The checked nodes :" + checkNodes);
      let catId = [];
      for (let node of checkNodes) {
        catId.push(node.catId);
      }
      console.log(catId);

      this.$confirm("确认批量删除分类?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "delete",
          data: this.$http.adornData(catId, false),
        }).then(({ data }) => {
          if (data.code === 0) {
            this.$message({
              showClose: true,
              message: "删除成功",
              type: "success",
            });
          } else {
            this.$message({
              message: "删除失败！" + data.msg,
              type: "error",
            });
          }
          this.getMenus(); // 刷新菜单
        });
      });
    },
  },

  created() {
    // 窗口载入时候就获取菜单列表
    this.getMenus();
  },
};
</script>

<style>
</style>