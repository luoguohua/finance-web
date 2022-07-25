<template>
  <div class="app-container">
    <div class="filter-container">
      <el-input
        v-model="queryParams.ledgerName"
        placeholder="请输入账本关键字"
        class="filter-item search-item"
        @change="search"
      />
      <!--<el-button icon="el-icon-search" >搜索</el-button>-->
      <el-button type="primary" icon="el-icon-plus" @click="add">
        新增
      </el-button>
    </div>
    <el-row v-loading="loading" :gutter="30">
      <el-col v-for="(item, index) in list" :key="index" :span="6">
        <el-card :body-style="{ padding: '0px' }" style="margin-bottom: 30px;">
          <el-image
            :src="require('@/assets/images/book.png')"
            style="width: 100%"
          />
          <div style="padding: 14px;font-size: 16px;">
            <span>{{ item.ledgerName }}</span>
            <div class="bottom" style="margin: 5px 0px;">
              <time class="time">{{ item.createTime }}</time>
            </div>
            <el-button
              type="text"
              class="text-button"
              size="mini"
              style="font-size: 16px"
              @click="update"
            >
              修改
            </el-button>
            <el-button
              type="text"
              class="text-button"
              size="mini"
              style="font-size: 16px"
              @click="deleted"
            >
              删除
            </el-button>
          </div>
        </el-card>
      </el-col>
    </el-row>
    <ledger-edit
      ref="edit"
      :dialog-visible="dialog.isVisible"
      :title="dialog.title"
      @success="editSuccess"
      @close="editClose"
    />
  </div>
</template>
<script>
import LedgerEdit from "./Edit";

export default {
  name: "MyLedger",
  components: { LedgerEdit },
  data() {
    return {
      list: [],
      loading: false,
      queryParams: {},
      form: {},
      dialog: {
        isVisible: false,
        title: ""
      }
    };
  },
  mounted() {
    this.fetch();
  },
  methods: {
    fetch(params = {}) {
      this.loading = true;
      this.$get("ledger", { ...params }).then(r => {
        const data = r.data.data;
        this.list = data;
        this.loading = false;
      });
    },
    search() {
      this.fetch({
        ...this.queryParams
      });
    },
    add() {
      this.dialog.title = "新增账本";
      this.dialog.isVisible = true;
    },
    update() {
      console.log("修改");
    },
    deleted() {
      console.log("删除");
    },
    editClose() {
      this.dialog.isVisible = false;
    },
    editSuccess() {
      this.search();
    }
  }
};
</script>
<style>
.text-button {
  padding: 0 !important;
}

.filter-container {
  display: flex;
  align-items: baseline;
}
.el-dialog {
  display: flex;
  flex-direction: column;
  margin: 0 !important;
  position: absolute;
  top: 40%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.el-dialog .el-dialog__body {
  flex: 1;
  overflow: auto;
}
</style>
