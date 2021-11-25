<template>
  <div class="userDetails">
    <el-card>
      <div v-loading="isLoading" class="main-area">
        <el-table
          ref="multipleTableRef"
          v-loading="isLoading"
          :data="tableData"
          highlight-current-row
          style="width: 100%"
          @select="handleSelect"
          @select-all="handleSelectAll">
          <el-table-column type="selection" width="55"></el-table-column>
          <el-table-column label="菜单-1" prop="name"></el-table-column>
          <el-table-column label="菜单-2" prop="clientId"></el-table-column>
          <el-table-column label="菜单-3" prop="belongSaleArea"></el-table-column>
          <el-table-column label="菜单-4" prop="sourceChannel"></el-table-column>
          <el-table-column label="菜单-5" prop="inboundOrderCount"></el-table-column>
          <el-table-column label="菜单-6" prop="inboundUsd"></el-table-column>
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
  name: 'userDetails',
  data() {
    return {
      isLoading: false,
      isDisabled: false,
      tableData: [],
      selectionList: [],
      cacheSelectIds: [],
      pagination: {
        total: 0,
        pageSize: 10,
        currentPage: 1,
        layout: 'prev, pager, next',
      },
      classifyInfo: '',
    }
  },
  created() {

    this.getList()
  },
  mounted() {
  },
  methods: {
    async getList() {
      try {
        const data = {
          page: this.pagination.currentPage,
          size: this.pagination.pageSize,
          userType: this.classifyInfo.name,
          ...JSON.parse(this.classifyInfo.overviewReq),
          // userType: '工厂'
        }

        this.isLoading = true

        this.tableData = [...Array(10)].map((k, i) => {
          return {
            id: `${ i }_${ this.pagination.currentPage }`,
            selection: 'selection' + i,
            name: `${ i }_${ this.pagination.currentPage }`,
            clientId: 'clientId' + i,
            belongSaleArea: 'belongSaleArea' + i,
            sourceChannel: 'sourceChannel' + i,
            inboundOrderCount: 'inboundOrderCount' + i,
            inboundUsd: 'inboundUsd' + i,
          }
        })

        this.pagination.total = 50

        this.toggleSelection()


      } catch (e) {
        console.log(e)
      } finally {
        this.isLoading = false
      }
    },
    handleCurrentChange(p) {
      this.pagination.currentPage = p
      this.getList()
    },
    handleSelectAll(selection) {
      // 取消全选，删除缓存里面的ID
      if (!selection.length) {
        const currentTableIds = this.tableData.map(k => k.id)

        currentTableIds.forEach((id, i) => {
          this.delCacheSelectId(id)
        })
      }

      const selectionId = selection.map(k => k.id)
      this.cacheSelectIds = Array.from(new Set([...this.cacheSelectIds, ...selectionId]))
    },
    handleSelect(selection, row) {
      const selectionId = selection.map(k => k.id)
      // 判断是否删除
      if (selectionId.some(id => id === row.id)) {
        this.cacheSelectIds.push(row.id)
      } else {
        this.delCacheSelectId(row.id)
      }
    },
    delCacheSelectId(id) {
      const findIndex = this.cacheSelectIds.findIndex(cacheId => cacheId === id)
      this.cacheSelectIds.splice(findIndex, 1)
    },
    toggleSelection() {
      // 需要异步更新
      this.$nextTick(() => {
        this.tableData.forEach((item, i) => {
          if (this.cacheSelectIds.some(cacheId => cacheId === item.id)) {
            this.$refs['multipleTableRef'].toggleRowSelection(this.tableData[i])
          }
        })
      })
    },
  },
}
</script>

<style lang="scss" scoped>
.userDetails {
  padding: 24px;
  padding-top: 0px;

  .title-p1 {
    margin: 16px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    font-family: Helvetica Neue;
    font-size: 20px;

    .btn-box {
      font-size: 14px;
      color: rgba(0, 0, 0, 0.45);
    }
  }

  .pagination-box {
    margin-top: 30px;
    display: flex;
    justify-content: flex-end;
  }
}
</style>
