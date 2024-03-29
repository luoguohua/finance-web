<template>
  <el-dialog :title="title"
             :width="width"
             top="50px"
             :close-on-click-modal="false"
             :close-on-press-escape="false"
             :visible.sync="isVisible"
  >
    <el-form ref="form"
             :model="ledger"
             :rules="rules"
             label-position="right"
             label-width="120px"
    >
      <el-form-item label="账本名称："
                    prop="ledgerName"
      >
        <el-input v-model="ledger.ledgerName" />
      </el-form-item>
      <el-form-item label="评估月收入："
                    prop="assessedMonthlyIncome"
      >
        <el-input-number v-model="ledger.assessedMonthlyIncome"
                         :step="1000"
                         :min="0"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary"
                   icon="el-icon-plus"
                   class="form-button"
                   size="mini"
                   @click="showBudgetDialog()"
        >
          新增
        </el-button>
        <el-table :data="ledger.budgetCategories"
                  border
                  style="width: 100%"
        >
          <el-table-column prop="categoryName"
                           label="预算分类名称"
          />
          <el-table-column prop="categoryCode"
                           label="预算分类编码"
          />
          <el-table-column prop="budgetProportion"
                           label="预算占比"
          />
          <el-table-column prop="budgetAmounts"
                           label="预算金额"
          />
          <el-table-column fixed="right"
                           label="操作"
                           width="150"
          >
            <template slot-scope="scope">
              <el-button type="text"
                         size="small"
                         class="table-opt-button"
                         @click="edit(scope.row)"
              >
                编辑
              </el-button>
              <el-button type="text"
                         size="small"
                         class="table-opt-button"
                         @click="remove(scope.row)"
              >
                删除
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
    </el-form>
    <div slot="footer"
         class="dialog-footer"
    >
      <el-button type="warning"
                 plain
                 :loading="buttonLoading"
                 @click="isVisible = false"
      >
        {{ $t("common.cancel") }}
      </el-button>
      <el-button type="primary"
                 plain
                 :loading="buttonLoading"
                 @click="submitForm"
      >
        {{ $t("common.confirm") }}
      </el-button>
    </div>
    <div z-index="999">
      <el-dialog v-if="budgetFormDialog"
                 :title="budgetTitle"
                 :visible.sync="budgetFormDialog"
                 append-to-body
      >
        <el-form ref="itemForm"
                 :model="budgetItem"
                 :rules="itemRules"
        >
          <el-form-item label="预算分类名称"
                        :label-width="formLabelWidth"
                        prop="categoryName"
          >
            <el-input v-model="budgetItem.categoryName"
                      autocomplete="off"
            />
          </el-form-item>
          <el-form-item label="预算分类编码"
                        :label-width="formLabelWidth"
                        prop="categoryCode"
          >
            <el-input v-model="budgetItem.categoryCode"
                      autocomplete="off"
                      :disabled="budgetItem.id"
            />
          </el-form-item>
          <el-form-item label="预算分类占比（%）"
                        :label-width="formLabelWidth"
                        prop="budgetProportion"
          >
            <el-input-number v-model="budgetItem.budgetProportion"
                             :min="0"
                             :max="100"
                             @change="calculateAmount"
            />
          </el-form-item>
          <el-form-item label="预算金额（元）"
                        :label-width="formLabelWidth"
                        prop="budgetAmounts"
          >
            <el-input-number v-model="budgetItem.budgetAmounts"
                             :max="9999999"
                             :precision="2"
                             :disabled="true"
            />
          </el-form-item>
        </el-form>
        <div slot="footer"
             class="dialog-footer"
        >
          <el-button @click="closeItem">
            取 消
          </el-button>
          <el-button type="primary"
                     @click="addBudget"
          >
            确 定
          </el-button>
        </div>
      </el-dialog>
    </div>
  </el-dialog>
</template>
<script>
import "@riophae/vue-treeselect/dist/vue-treeselect.css";

export default {
  name: "LedgerEdit",
  props: {
    dialogVisible: {
      type: Boolean,
      default: false
    },
    title: {
      type: String,
      default: ""
    }
  },
  data() {
    return {
      ledger: this.initLedger(),
      width: this.initWidth(),
      budgetItem: {},
      budgetTitle: "",
      budgetFormDialog: false,
      formLabelWidth: "150px",
      screenWidth: 0,
      rules: {
        ledgerName: [
          { required: true, message: "请输入账本名称", trigger: "blur" }
        ],
        assessedMonthlyIncome: [
          {
            type: "number",
            required: true,
            message: "请输入评估月收入",
            trigger: "change"
          }
        ]
      },
      buttonLoading: false,
      itemRules: {
        categoryName: [
          { required: true, message: "请输入分类名称", trigger: "blur" }
        ],
        categoryCode: [
          { required: true, message: "请输入分类编码", trigger: "blur" }
        ],
        budgetProportion: [
          {
            type: "number",
            required: true,
            message: "请输入分类占比",
            trigger: "change",
            max: 100,
            min: 0
          }
        ]
      }
    };
  },
  computed: {
    isVisible: {
      get() {
        return this.dialogVisible;
      },
      set() {
        this.close();
        this.reset();
      }
    }
  },
  mounted() { },
  methods: {
    submitForm() {
      this.$refs.form.validate((valid) => {
        if (valid) {
          console.log('提交')
          this.$post('ledger', JSON.stringify(this.ledger)).then(() => {
            // this.buttonLoading = false
            // this.isVisible = false
            // this.$message({
            //   message: this.$t('tips.createSuccess'),
            //   type: 'success'
            // })
            // this.$emit('success')
          })
        }
      });
    },
    initLedger() {
      return {
        ledgerName: "",
        assessedMonthlyIncome: 0,
        budgetCategories: []
      };
    },
    initWidth() {
      this.screenWidth = document.body.clientWidth;
      if (this.screenWidth < 991) {
        return "90%";
      } else if (this.screenWidth < 1400) {
        return "45%";
      } else {
        return "800px";
      }
    },
    close() {
      this.$emit("close");
    },
    reset() {
      // 先清除校验，再清除表单，不然有奇怪的bug
      this.$refs.form.clearValidate();
      this.$refs.form.resetFields();
      this.ledger = this.initLedger();
    },
    closeItem() {
      // 先清除校验，再清除表单，不然有奇怪的bug
      this.$nextTick(() => {
        this.budgetFormDialog = false
        this.$refs.itemForm.clearValidate();
        this.$refs.itemForm.resetFields();
        this.budgetItem = {};
      })
    },
    remove(row) {
      this.$nextTick(() => {
        this.budgetItem = row;
        this.excludeData();
        this.budgetItem = {};
      })
    },
    edit(row) {
      this.budgetTitle = "修改预算类目";
      this.$nextTick(() => {
        this.budgetFormDialog = true;
        this.budgetItem = row;
      })
    },
    showBudgetDialog() {
      this.budgetTitle = "新预算类目";
      this.budgetFormDialog = true;
    },
    addBudget() {
      this.$refs.itemForm.validate((valid) => {
        if (valid) {
          this.budgetFormDialog = false;
          this.excludeData()
          this.budgetItem.key = new Date().getTime();
          this.ledger.budgetCategories.push(this.budgetItem);
          this.budgetItem = {};
        }
      });
    },
    excludeData() {
      if (this.budgetItem.id) {
        this.ledger.budgetCategories = this.ledger.budgetCategories.filter(item => {
          return item.id !== this.budgetItem.id;
        })
      } else if (this.budgetItem.key) {
        this.ledger.budgetCategories = this.ledger.budgetCategories.filter(item => {
          return item.key !== this.budgetItem.key;
        })
      }
    },
    calculateAmount() {
      this.$nextTick(() => {
        this.budgetItem.budgetAmounts = this.ledger.assessedMonthlyIncome * (this.budgetItem.budgetProportion / 100)
      })
    }
  }
};
</script>
<style lang="scss" scoped>
.table-opt-button {
  padding: 0px 0px !important;
}
.form-button {
  padding: 7px 15px !important;
  position: relative;
  left: 88.3%;
}
</style>
