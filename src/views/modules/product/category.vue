<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    :expand-on-click-node="false"
    show-checkbox
    node-key="catId"
    :default-expanded-keys="expandedKey"
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
</template>

<script>
export default {
  componets: {},
  props: {},
  data() {
    return {
      menus: [],
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
        console.log(that.menus);
      });
    },

    append(data) {},

    remove(node, data) {
      let that = this;
      console.log("node：" + node);
      this.$confirm("确认删除该分类: " + data.name + "?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(() => {

          let ids = [data.catId];

          this.$http({

            url: this.$http.adornUrl("/product/category/delete"),
            method: "delete",
            data: this.$http.adornData(ids, false),

          }).then((res) => {
            console.log("ids = " + ids);
            console.log("delete successfully");
            console.log(res);
            that.getMenus();// 更新删除操作后的数据
          });

          this.$message({ // 删除成功弹窗
            type: "success", 
            message: "删除分类成功: " + data.name,
          });

          this.expandedKey = [node.parent.data.catId]; // 删除后依然展开删去节点的父节点

        })
        .catch(() => { // 取消删除操作

          this.$message({ // 弹窗
            type: "info",
            message: "已取消删除",
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