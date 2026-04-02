<template>
  <div class="pane-container">
    <!-- 工具栏 -->
    <div class="toolbar">
      <el-button type="primary" @click="showHandoverDialog">
        <i class="el-icon-plus"></i>新增交接记录
      </el-button>
    </div>

    <!-- 列表 -->
    <el-table
        :data="list"
        border
        v-loading="loading"
        stripe
        style="width: 100%"
    >
      <el-table-column label="交接人" prop="handoverByName" width="120" />
      <el-table-column label="接收人" prop="receiveByName" width="120" />
      <el-table-column label="交接时间" prop="handoverTime" width="180" />
      <el-table-column label="交接物品" min-width="200">
        <template slot-scope="{ row }">
          {{ row.itemNames.replace(/,/g, '、') || '无' }}
        </template>
      </el-table-column>
      <el-table-column label="备注" prop="remark" min-width="200" />
      <el-table-column label="操作" width="100">
        <template slot-scope="{ row }">
          <el-button
              type="danger"
              style="padding: 3px"
              @click="handleDelete(row)"
              v-if="isAllowDelete(row)"
          >
            删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <div class="pagination-box">
      <el-pagination
          background
          @current-change="handlePageChange"
          @size-change="handleSizeChange"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          :page-sizes="[10, 20, 30]"
          :current-page="page"
          :page-size="size"
      />
    </div>

    <!-- 新增弹窗 -->
    <el-dialog
        title="新增交接记录"
        center
        :visible.sync="dialogVisible"
        width="40%"
        :close-on-click-modal="false"
    >
      <el-form
          :model="form"
          :rules="rules"
          ref="handoverForm"
          label-width="110px"
      >
        <el-form-item label="交接人" prop="handoverBy">
          <el-select
              v-model="form.handoverBy"
              placeholder="请搜索并选择交接人"
              style="width: 280px"
              filterable
              clearable
          >
            <el-option
                v-for="item in employeeList"
                :key="item.id"
                :label="`${item.name} | ${item.department.name} | ${item.position.name}`"
                :value="item.id"
            />
          </el-select>
        </el-form-item>

        <el-form-item label="接收人" prop="receiveBy">
          <el-select
              v-model="form.receiveBy"
              placeholder="请搜索并选择接收人"
              style="width: 280px"
              filterable
              clearable
          >
            <el-option
                v-for="item in employeeList"
                :key="item.id"
                :label="`${item.name} | ${item.department.name} | ${item.position.name}`"
                :value="item.id"
            />
          </el-select>
        </el-form-item>

        <el-form-item label="交接时间" prop="handoverTime">
          <el-date-picker
              v-model="form.handoverTime"
              type="datetime"
              style="width: 100%"
              value-format="yyyy-MM-dd HH:mm:ss"
              placeholder="请选择交接时间"
          />
        </el-form-item>

        <el-form-item label="备注">
          <el-input
              v-model="form.remark"
              type="textarea"
              rows="3"
              style="width: 100%"
              placeholder="请输入备注信息"
          />
        </el-form-item>

        <el-divider>交接物品</el-divider>
        <el-checkbox-group v-model="form.itemNames" class="checkbox-group">
          <el-checkbox label="工服" />
          <el-checkbox label="发票本" />
          <el-checkbox label="对讲机" />
          <el-checkbox label="开酒器" />
          <el-checkbox label="宿舍钥匙" />
          <el-checkbox label="宿舍用品三件套" />
        </el-checkbox-group>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submitHandover">确认保存</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "HandoverRecord",
  data() {
    return {
      // 列表数据
      list: [],
      total: 0,
      page: 1,
      size: 10,
      loading: false,
      empId:"",
      empName:"",
      // 弹窗
      dialogVisible: false,

      // 员工选择列表
      employeeList: [],

      // 表单数据
      form: {
        handoverBy: "",
        receiveBy: "",
        handoverTime: "",
        remark: "",
        createBy: "",       // 新增：创建人ID
        createByName: "",
        itemNames: []
      },

      // 校验规则
      rules: {
        handoverBy: [
          { required: true, message: "请选择交接人", trigger: "change" }
        ],
        receiveBy: [
          { required: true, message: "请选择接收人", trigger: "change" }
        ],
        handoverTime: [
          { required: true, message: "请选择交接时间", trigger: "change" }
        ]
      }
    };
  },
  mounted() {
    // 初始化加载
    this.initPage();
  },
  methods: {
    /**
     * 页面初始化
     */
    initPage() {
      this.loadEmployeeList();
      this.getHandoverList();
    },

    /**
     * 加载员工列表
     */
    loadEmployeeList() {
      this.getRequest("/employee/basic/employees").then(res => {
        this.employeeList = res || [];
      });
    },

    /**
     * 获取交接记录列表
     */
    getHandoverList() {
      let user = JSON.parse(sessionStorage.getItem('user'));
      this.empId = user.employeeId;
      this.empName = user.name;
      this.loading = true;
      this.getRequest(`/employee/handover/?page=${this.page}&size=${this.size}&empId=`+(user.employeeId || '')).then(res => {
        this.list = res.data;
        this.total = res.total;
        this.loading = false;
      });
    },

    // 删除权限判断
    isAllowDelete(row) {
      if (!this.empId) {
        return true;
      }
      return row.createBy === this.empId ;
    },

    // 删除
    handleDelete(row) {
      if (!this.isAllowDelete(row)) {
        this.$message.error("无删除权限");
        return;
      }
      this.$confirm("确定删除该交接记录？", "提示", { type: "warning" })
          .then(() => {
            this.deleteRequest("/employee/handover/" + row.id).then(() => {
              this.$message.success("删除成功");
              this.getHandoverList();
            });
          })
          .catch(() => {});
    },
    /**
     * 打开新增弹窗
     */
    showHandoverDialog() {
      // 重置表单
      this.resetForm();
      this.dialogVisible = true;

      // 清除校验
      this.$nextTick(() => {
        if (this.$refs.handoverForm) {
          this.$refs.handoverForm.clearValidate();
        }
      });
    },

    /**
     * 重置表单
     */
    resetForm() {
      this.form = {
        handoverBy: "",
        receiveBy: "",
        handoverTime: "",
        remark: "",
        createBy: "",       // 新增：创建人ID
        createByName: "",
        itemNames: []
      };
    },

    /**
     * 提交交接记录
     */
    submitHandover() {
      this.$refs.handoverForm.validate(valid => {
        if (!valid) return;

        // 交接人和接收人不能相同
        if (this.form.handoverBy === this.form.receiveBy) {
          this.$message.warning("交接人和接收人不能为同一人");
          return;
        }
        if (this.form.itemNames.length === 0) {
          this.$message.warning("请至少选择一项交接物品！");
          return;
        }

        this.form.createBy = this.empId;
        this.form.createByName = this.empName;

        this.form.itemNames = this.form.itemNames.join(',');
        this.loading = true;
        this.postRequest("/employee/handover/add", this.form)
            .then(() => {
              this.$message.success("交接记录保存成功");
              this.dialogVisible = false;
              this.getHandoverList(); // 刷新列表
            })
            .finally(() => {
              this.loading = false;
            });
      });
    },

    /**
     * 页码改变
     */
    handlePageChange(val) {
      this.page = val;
      this.getHandoverList();
    },

    /**
     * 每页条数改变
     */
    handleSizeChange(val) {
      this.size = val;
      this.page = 1;
      this.getHandoverList();
    }
  }
};
</script>

<style scoped>
.pane-container {
  padding: 20px;
  background: #fff;
  min-height: calc(100vh - 60px);
}

.toolbar {
  margin-bottom: 15px;
}

.pagination-box {
  margin-top: 15px;
  text-align: right;
}

.checkbox-group {
  display: flex;
  flex-wrap: wrap;
  gap: 18px 25px;
  padding: 5px 0;
}

.dialog-footer {
  text-align: right;
}
</style>