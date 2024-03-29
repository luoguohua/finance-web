<template>
  <div class="app-container">
    <el-alert
      style="margin: -1rem 0 1.2rem 0;padding: 1rem;"
      :title="$t('table.blackList.tips')"
      type="info"
      :closable="false"
    />
    <div class="filter-container">
      <el-input v-model="queryParams.ip" :placeholder="$t('table.blackList.ip')" class="filter-item search-item" />
      <el-input v-model="queryParams.requestUri" :placeholder="$t('table.blackList.requestUri')" class="filter-item search-item" />
      <el-input v-model="queryParams.requestMethod" :placeholder="$t('table.blackList.requestMethod')" class="filter-item search-item" />
      <el-button class="filter-item" type="primary" plain @click="search">
        {{ $t('table.search') }}
      </el-button>
      <el-button class="filter-item" type="success" plain @click="reset">
        {{ $t('table.reset') }}
      </el-button>
      <el-dropdown trigger="click" class="filter-item">
        <el-button>
          {{ $t('table.more') }}<i class="el-icon-arrow-down el-icon--right" />
        </el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item @click.native="add">{{ $t('table.add') }}</el-dropdown-item>
          <el-dropdown-item @click.native="batchDelete">{{ $t('table.delete') }}</el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
    </div>
    <el-table
      ref="table"
      :key="tableKey"
      v-loading="loading"
      :data="list"
      border
      fit
      style="width: 100%;"
      @selection-change="onSelectChange"
      @sort-change="sortChange"
    >
      <el-table-column type="selection" align="center" width="40px" />
      <el-table-column :label="$t('table.blackList.ip')" prop="ip" :show-overflow-tooltip="true" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.ip ? scope.row.ip : $t('table.blackList.allIp') }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.blackList.requestUri')" prop="requestUri" :show-overflow-tooltip="true" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.requestUri }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.blackList.requestMethod')" prop="requestMethod" :show-overflow-tooltip="true" align="center">
        <template slot-scope="{row}">
          <el-tag :type="row.requestMethod | requestMethodFilter">
            {{ row.requestMethod }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.blackList.limitFrom')" prop="limitFrom" :show-overflow-tooltip="true" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.limitFrom ? scope.row.limitFrom : $t('table.blackList.allTheTime') }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.blackList.limitTo')" prop="limitTo" :show-overflow-tooltip="true" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.limitTo ? scope.row.limitTo : $t('table.blackList.allTheTime') }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.blackList.location')" prop="location" :show-overflow-tooltip="true" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.location }}</span>
        </template>
      </el-table-column>
      <el-table-column
        :label="$t('table.blackList.status')"
        :filters="[{ text: $t('common.status.valid'), value: '1' }, { text: $t('common.status.invalid'), value: '0' }]"
        :filter-method="filterStatus"
        class-name="status-col"
      >
        <template slot-scope="{row}">
          <el-tag :type="row.status | statusFilter">
            {{ row.status === '1' ? $t('common.status.valid') : $t('common.status.invalid') }}
          </el-tag>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.blackList.createTime')" prop="createTime" :show-overflow-tooltip="true" align="center" sortable="custom">
        <template slot-scope="scope">
          <span>{{ scope.row.createTime }}</span>
        </template>
      </el-table-column>
      <el-table-column :label="$t('table.operation')" align="center" min-width="60px" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <i class="el-icon-setting table-operation" style="color: #2db7f5;" @click="edit(row)" />
          <i class="el-icon-delete table-operation" style="color: #f50;" @click="singleDelete(row)" />
        </template>
      </el-table-column>
    </el-table>
    <pagination v-show="total>0" :total="total" :page.sync="pagination.num" :limit.sync="pagination.size" @pagination="search" />
    <BlackListEdit
      ref="edit"
      :dialog-visible="dialog.isVisible"
      :title="dialog.title"
      @success="editSuccess"
      @close="editClose"
    />
  </div>
</template>
<script>
import r from '@/utils/route-request'
import axios from 'axios'
import Pagination from '@/components/Pagination'
import BlackListEdit from './Edit'

export default {
  name: 'BlackList',
  components: { Pagination, BlackListEdit },
  filters: {
    requestMethodFilter(v) {
      if (v === 'GET') {
        return 'success'
      } else if (v === 'POST') {
        return ''
      } else if (v === 'DELETE') {
        return 'danger'
      } else if (v === 'PUT') {
        return 'warning'
      } else {
        return 'info'
      }
    },
    statusFilter(status) {
      const map = {
        0: 'danger',
        1: 'success'
      }
      return map[status]
    }
  },
  data() {
    return {
      tableKey: 0,
      loading: false,
      dialog: {
        isVisible: false,
        title: ''
      },
      list: null,
      total: 0,
      queryParams: {},
      sort: {},
      selection: [],
      pagination: {
        size: 10,
        num: 1
      }
    }
  },
  mounted() {
    this.fetch()
  },
  methods: {
    filterStatus(value, row) {
      return row.status === value
    },
    editClose() {
      this.dialog.isVisible = false
    },
    editSuccess() {
      this.search()
    },
    add() {
      this.dialog.title = this.$t('common.add')
      this.dialog.isVisible = true
    },
    edit(row) {
      if (row.limitFrom && row.limitTo) {
        row.timeLimit = '1'
      }
      this.$refs.edit.setblackList(row)
      this.dialog.title = this.$t('common.edit')
      this.dialog.isVisible = true
    },
    batchDelete() {
      if (!this.selection.length) {
        this.$message({
          message: this.$t('tips.noDataSelected'),
          type: 'warning'
        })
        return
      }
      this.$confirm(this.$t('tips.confirmDelete'), this.$t('common.tips'), {
        confirmButtonText: this.$t('common.confirm'),
        cancelButtonText: this.$t('common.cancel'),
        type: 'warning'
      }).then(() => {
        const blackListIds = []
        this.selection.forEach((r) => {
          blackListIds.push(r.id)
        })
        this.delete(blackListIds)
      }).catch(() => {
        this.clearSelections()
      })
    },
    clearSelections() {
      this.$refs.table.clearSelection()
    },
    singleDelete(row) {
      this.$refs.table.toggleRowSelection(row, true)
      this.batchDelete()
    },
    delete(blackListIds) {
      this.loading = true
      r.delete('route/blackList', { ids: blackListIds }).then(() => {
        this.$message({
          message: this.$t('tips.deleteSuccess'),
          type: 'success'
        })
        this.search()
      }).catch(r => {
        this.loading = false
      })
    },
    search() {
      this.fetch({
        ...this.queryParams,
        ...this.sort
      })
    },
    reset() {
      this.queryParams = {}
      this.sort = {}
      this.$refs.table.clearSort()
      this.$refs.table.clearFilter()
      this.search()
    },
    onSelectChange(selection) {
      this.selection = selection
    },
    sortChange(val) {
      this.sort.field = val.prop
      this.sort.order = val.order
      this.search()
    },
    fetch(params = {}) {
      params.pageSize = this.pagination.size
      params.pageNum = this.pagination.num - 1
      this.loading = true
      axios.all([
        r.get('route/blackList/data', { ...params }),
        r.get('route/blackList/count', { ...params })
      ]).then(v => {
        this.total = v[1].data
        this.list = v[0].data
        this.loading = false
      }).catch(r => {
        this.loading = false
      })
    }
  }
}
</script>
