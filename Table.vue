<template>
  <div id="table">
    <!-- 表格 -->
    <div class="excel">
      <download-excel :data="dataSource" :fields="fields" :name="excelName">
        <el-button type="primary">导出数据</el-button>
      </download-excel>
    </div>
    <el-button class="export" type="info" @click="addData" v-if="showAddButton"
      >添加</el-button
    >
    <el-button class="search" type="info" @click="search">搜索</el-button>
    <el-input
      class="title_input"
      :placeholder="searchProps.prompt"
      v-model="keyWord"
      @keyup.enter.native="search"
    ></el-input>
    <el-table
      class="el-tb-edit"
      :row-style="rowClassName"
      :empty-text="emptyText"
      @current-change="handleCurrentRowChange"
      highlight-current-row
      :data="
        dataSource.slice((currentPage - 1) * pagesize, currentPage * pagesize)
      "
      v-loading="loading"
      element-loading-text="正在加载"
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(0, 0, 0, 0.8)"
      ref="tableRef"
    >
      <!--数据源-->
      <el-table-column
        v-for="(column, row) in columns"
        header-align="center"
        :sortable="column.hasSort"
        :key="column.prop"
        :prop="column.prop"
        :label="column.label"
        :align="column.align"
        :width="column.width"
      >
        <template slot-scope="{ row, $index }">
          <span>{{ row[column.prop] }}</span>
        </template>
      </el-table-column>

      <el-table-column label="操作" align="center" width="400">
        <template slot-scope="{ row, $index }">
          <el-button
            v-for="button in buttons"
            :key="button.id"
            size="small"
            :type="button.type"
            @click="button.operate($index, row)"
            >{{ button.label }}</el-button
          >
          <!-- <el-button size="small" type="warning" @click="edit($index, row)">编辑</el-button>
                  <el-button size="small" type="danger" @click="handleDelete($index, row)">删除</el-button>
                  <el-button size="small" type="danger" @click="detail($index, row)">详情</el-button> -->
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
      class="pagination"
      background
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage"
      :page-sizes="[5, 10, 20, 50]"
      :page-size="pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
    >
    </el-pagination>
  </div>
</template>

<script>
export default {
  name: "tables",
  props: {
    dataSource: {
      // 表格数据源
      type: Array,
      default: () => [],
    },
    columns: {
      // 表格的字段展示
      type: Array,
      default: () => [],
    },

    buttons: {
      type: Array,
      default: () => [],
    },

    rowClassName: {
      type: Function,
    },
    searchProps: {
      type: Object,
    },
    loading: {
      type: Boolean,
      default: true,
    },
    initTableData: {
      type: Array,
    },
    addData: {
      type: Function,
    },
    emptyText: {
      type: String,
      default: "暂无数据",
    },
    excelName: {
      type: String,
      default: "信息表",
    },
    showAddButton: {
      type: Boolean,
      default: true,
    },
  },
  data() {
    return {
      keyWord: "",

      currentPage: 1,
      table_index: 999,
      pagesize: 5,
      currentRow: null,
      title: "",
    };
  },

  computed: {
    total() {
      return this.dataSource.length;
    },
    fields() {
      var filedsJson = {};
      for (var i = 0; i < this.columns.length; i++) {
        this.$set(filedsJson, this.columns[i].label, this.columns[i].prop);
      }

      return filedsJson;
    },
  },
  created() {},
  methods: {
    create() {},
    search() {
      var prop = this.searchProps.prop;
      var keyWord = this.keyWord;
      if (keyWord == "") {
        this.$message("搜索内容不能为空！");
        return;
      }

      var temp = [];
      // console.log(this.initTableData);
      temp = this.initTableData.filter(
        (item) => item[prop].search(keyWord) != -1
      );
      // console.log(temp);
      this.$emit("changeTableData", temp);
    },

    handleCurrentRowChange(val) {
      this.currentRow = val;
      //   console.log(this.$refs);
    },
    handleEdit(index, row) {
      console.log(index, row);
    },

    handleSizeChange(size) {
      this.pagesize = size;
    },
    handleCurrentChange(currentPage) {
      this.currentPage = currentPage;
    },
  },
};
</script>

<style scoped>
.export {
  margin: 20px;
  float: left;
}

.excel {
  margin: 20px;
  float: left;
}

.title_input {
  margin: 20px;
  width: 20%;
  float: right;
}
.search {
  margin: 20px;
  float: right;
}
</style>


