<template>
  <div class="EleTable">
    <el-card>
      <div v-loading="isLoading">
        <el-table
          ref="multipleTableRef"
          v-loading="isLoading"
          :data="tableData"
          highlight-current-row
          style="width: 100%"
          @select="handleSelect"
          @select-all="handleSelectAll">
          <el-table-column type="selection" width="55"></el-table-column>
          <el-table-column label="菜单-1" prop="col-1"></el-table-column>
          <el-table-column label="菜单-2" prop="col-2"></el-table-column>
          <el-table-column label="菜单-3" prop="col-3"></el-table-column>
          <el-table-column label="菜单-4" prop="col-4"></el-table-column>
          <el-table-column label="菜单-5" prop="col-5"></el-table-column>
        </el-table>
      </div>
      <div class="pagination-box">
        <el-pagination
          v-if="pagination.total > pagination.pageSize"
          background
          v-bind="pagination"
          @current-change="handleCurrentChange"></el-pagination>
      </div>
    </el-card>
  </div>
</template>

<script>
export default {
  name: 'EleTable',
  data() {
    return {
      isLoading: false,
      tableData: [],
      selectionList: [],
      cacheSelectIds: [],
      pagination: {
        total: 0,
        pageSize: 10,
        currentPage: 1,
        layout: 'prev, pager, next',
      },
    };
  },
  created() {
    this.getList();
  },
  methods: {
    async getList() {
      try {
        this.isLoading = true;
        this.tableData = [...Array(10)].map((k, i) => {
          return {
            id: `${ i }_${ this.pagination.currentPage }`,
            'col-1': `col-1_${ i }`,
            'col-2': `col-2_${ i }`,
            'col-3': `col-3_${ i }`,
            'col-4': `col-4_${ i }`,
            'col-5': `col-5_${ i }`,
          };
        });

        this.pagination.total = 50;

        this.toggleSelection();
      } catch (e) {
        console.log(e);
      } finally {
        this.isLoading = false;
      }
    },
    handleCurrentChange(p) {
      this.pagination.currentPage = p;
      this.getList();
    },
    handleSelectAll(selection) {
      // 取消全选，删除缓存里面的ID
      if (!selection.length) {
        const currentTableIds = this.tableData.map(k => k.id);

        currentTableIds.forEach((id, i) => {
          this.delCacheSelectId(id);
        });
      }

      const selectionId = selection.map(k => k.id);
      this.cacheSelectIds = Array.from(new Set([...this.cacheSelectIds, ...selectionId]));
    },
    handleSelect(selection, row) {
      const selectionId = selection.map(k => k.id);
      // 判断是否删除
      if (selectionId.some(id => id === row.id)) {
        this.cacheSelectIds.push(row.id);
      } else {
        this.delCacheSelectId(row.id);
      }
    },
    delCacheSelectId(id) {
      const findIndex = this.cacheSelectIds.findIndex(cacheId => cacheId === id);
      this.cacheSelectIds.splice(findIndex, 1);
    },
    toggleSelection() {
      // 需要异步更新
      this.$nextTick(() => {
        this.tableData.forEach((item, i) => {
          if (this.cacheSelectIds.some(cacheId => cacheId === item.id)) {
            this.$refs['multipleTableRef'].toggleRowSelection(this.tableData[i]);
          }
        });
      });
    },
  },
};
</script>

<style lang="scss" scoped>
.pagination-box {
  margin-top: 30px;
  display: flex;
  justify-content: flex-end;
}
</style>
