<template>
  <div>
    <div class="top-style">
      <el-button type="primary" icon="el-icon-plus" @click="showApplyDialog">
        发起离职申请
      </el-button>
    </div>

    <!-- 离职申请弹窗 -->
    <el-dialog title="员工离职申请" center :visible.sync="dialogVisible" width="40%">
      <el-form :model="resign" :rules="rules" ref="resignForm" label-width="110px">
        <el-form-item label="申请人">
          <el-input v-model="resign.oldPosName" disabled style="width: 180px" />
        </el-form-item>

<!--        <el-form-item label="当前职称">
          <el-input v-model="resign.oldJobLevelName" disabled style="width: 180px" />
        </el-form-item>-->

        <el-form-item label="离职类型" prop="resignType">
          <el-select v-model="resign.resignType" placeholder="请选择离职类型" style="width: 180px">
            <el-option label="主动离职" :value="1" />
            <el-option label="被动离职" :value="2" />
          </el-select>
        </el-form-item>

        <el-form-item label="审批人" prop="approverId">
          <el-select
              v-model="resign.approverId"
              placeholder="请搜索并选择审批人"
              style="width: 280px"
              filterable
              clearable
          > <el-option
              v-for="item in approvers"
              :key="item.id"
              :label="`${item.name} | ${item.department.name} | ${item.position.name}`"
              :value="item.id"
          />
          </el-select>
        </el-form-item>

        <el-form-item label="期望离职日期" prop="expectResignDate">
          <el-date-picker
              v-model="resign.expectResignDate"
              type="date"
              placeholder="选择期望离职日期"
              style="width: 180px"
              value-format="yyyy-MM-dd"
          />
        </el-form-item>

        <el-form-item label="离职原因" prop="resignReason">
          <el-input
              type="textarea"
              v-model="resign.resignReason"
              maxlength="500"
              show-word-limit
              style="width: 350px"
              rows="3"
          />
        </el-form-item>
      </el-form>

      <div slot="footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submitResign">提交申请</el-button>
      </div>
    </el-dialog>

    <!-- 审批意见弹窗 -->
    <el-dialog title="审批意见" :visible.sync="approvalDialogVisible" width="30%">
      <el-form :model="approvalForm" label-width="80px">
        <el-form-item label="审批意见">
          <el-input
              v-model="approvalForm.remark"
              type="textarea"
              rows="5"
              placeholder="请填写审批意见（必填）"
          ></el-input>
        </el-form-item>
        <el-form-item label="实际离职日期" prop="actualResignDate" v-if="approvalForm.status === 1">
          <el-date-picker
              v-model="approvalForm.actualResignDate"
              type="date"
              placeholder="选择实际离职日期"
              style="width: 180px"
              value-format="yyyy-MM-dd"
          />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="approvalDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submitApproval">确认提交</el-button>
      </div>
    </el-dialog>

    <!-- 离职记录列表 -->
    <div class="content-style" style="margin-top:10px;">
      <el-table
          :data="resignList"
          border
          stripe
          max-height="680"
          v-loading="loading"
          style="width:100%"
      >
        <el-table-column label="申请人" prop="applyName" width="100"/>
        <el-table-column label="原职位" prop="oldPosName" width="100"/>
        <el-table-column label="离职类型" width="100">
          <template slot-scope="scope">
            {{ scope.row.resignType === 1 ? '主动离职' : '被动离职' }}
          </template>
        </el-table-column>
        <el-table-column label="期望离职日期" prop="expectResignDate" width="130"/>
        <el-table-column label="审批人" prop="approverName" width="100"/>
        <el-table-column label="离职原因" width="210" prop="resignReason" />
        <el-table-column label="申请时间" prop="createTime" width="160"/>
        <el-table-column label="审核时间" width="160">
          <template slot-scope="scope">
            {{ scope.row.approveTime || '-' }}
          </template>
        </el-table-column>
        <el-table-column label="实际离职日期" width="130">
          <template slot-scope="scope">
            {{ scope.row.actualResignDate || '-' }}
          </template>
        </el-table-column>
        <el-table-column label="审核意见" prop="approveRemark" width="210">
          <template slot-scope="scope">
            {{ scope.row.approveRemark || '-' }}
          </template>
        </el-table-column>
        <el-table-column label="审批状态" width="100">
          <template slot-scope="scope">
            <el-tag :type="scope.row.status === 0 ? 'warning' : scope.row.status === 1 ? 'success' : 'danger'">
              {{ scope.row.status === 0 ? '待审批' : scope.row.status === 1 ? '已通过' : '已拒绝' }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" fixed="right" width="120">
          <template slot-scope="scope">
            <!-- 我是审批人且状态待审批 -->
            <el-button
                v-if="scope.row.approverId === resign.empId && scope.row.status === 0"
                type="success"
                style="padding: 3px"
                @click="handlePreApprove(scope.row, 1)"
            >
              通过
            </el-button>
            <el-button
                v-if="scope.row.approverId === resign.empId && scope.row.status === 0"
                type="danger"
                style="padding: 3px"
                @click="handlePreApprove(scope.row, 2)"
            >
              拒绝
            </el-button>
            <!-- 我是申请人且状态待审批 -->
            <el-button
                v-else-if="scope.row.empId === resign.empId && scope.row.status === 0"
                type="danger"
                style="padding: 3px"
                @click="handleDelete(scope.row)"
            >
              删除
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div class="bottom-style">
      <el-pagination
          style="margin-top:10px"
          background
          @current-change="currentChange"
          @size-change="sizeChange"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          :page-sizes="[10, 20, 30]"
      />
    </div>
  </div>
</template>

<script>
export default {
  name: "ResignationApply",
  data() {
    return {
      resignList: [],
      loading: false,
      dialogVisible: false,
      total: 0,
      page: 1,
      size: 10,
      approvers: [],
      resign: {
        id: "",
        empId: "",
        oldPosId: "",
        oldPosName: "",
        oldJobLevelId: "",
        oldJobLevelName: "",
        resignType: "", // 1主动 2被动
        expectResignDate: "",
        resignReason: "",
        approverId: "",
        approverName: ""
      },
      approvalDialogVisible: false,
      approvalForm: {
        id: '',
        empId: '',
        status: '',
        remark: '',
        actualResignDate: ''
      },
      rules: {
        resignType: [{ required: true, message: "请选择离职类型", trigger: "change" }],
        approverId: [{ required: true, message: "请选择审批人", trigger: "change" }],
        expectResignDate: [{ required: true, message: "请选择期望离职日期", trigger: "change" }],
        resignReason: [{ required: true, message: "请填写离职原因", trigger: "blur" }]
      }
    };
  },
  mounted() {
    this.loadBaseData();
    this.getMyResignList();
  },
  methods: {
    // 打开审批弹窗
    handlePreApprove(row, status) {
      this.approvalForm.id = row.id;
      this.approvalForm.empId = row.empId;
      this.approvalForm.status = status;
      this.approvalForm.remark = '';
      this.approvalForm.actualResignDate = '';
      this.approvalDialogVisible = true;
    },

    // 提交审批
    submitApproval() {
      if (!this.approvalForm.remark) {
        this.$message.warning('请填写审批意见！');
        return;
      }
      if (this.approvalForm.status === 1 && !this.approvalForm.actualResignDate) {
        this.$message.warning('请选择实际离职日期！');
        return;
      }

      this.postRequest(`/employee/resignation/update`, {
        id: this.approvalForm.id,
        empId: this.approvalForm.empId,
        status: this.approvalForm.status,
        approveRemark: this.approvalForm.remark,
        actualResignDate: this.approvalForm.actualResignDate
      }).then(res => {
        this.$message.success('审核成功！');
        this.approvalDialogVisible = false;
        this.getMyResignList();
      });
    },

    handleDelete(row) {
      this.$confirm('确定要删除这条待审批的离职申请吗？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.deleteRequest("/employee/resignation/" + row.id).then(res => {
          this.$message.success('删除成功！');
          this.getMyResignList();
        });
      }).catch(() => {});
    },

    loadBaseData() {
      this.getRequest("/employee/basic/employees").then(res => this.approvers = res);
    },

    // 打开离职申请弹窗
    showApplyDialog() {
      const hasPendingApply = this.resignList.some(item =>
          item.empId === this.resign.empId && item.status === 0
      );
      if (hasPendingApply) {
        this.$message.warning('你已有待审批的离职申请，不可重复提交！');
        return;
      }

      this.getRequest('/employee/basic/findOne?id='+this.resign.empId).then(resp => {
        if (resp) {
          let emp = resp;
          this.resign.oldPosId = emp.position.id;
          this.resign.oldPosName = emp.position.name;
          this.resign.oldJobLevelId = emp.jobLevel.id;
          this.resign.oldJobLevelName = emp.jobLevel.name;
        }
      });
      this.dialogVisible = true;
    },

    // 提交离职申请
    submitResign() {
      this.$refs.resignForm.validate(valid => {
        if (valid) {
          if (!this.resign.resignReason.trim()) {
            this.$message.error('请填写离职原因！');
            return;
          }
          this.postRequest("/employee/resignation/add", this.resign).then(res => {
            this.$notify.success("离职申请提交成功！");
            this.dialogVisible = false;
            this.getMyResignList();
          });
        }
      });
    },

    // 获取我的离职记录
    getMyResignList() {
      let user = JSON.parse(sessionStorage.getItem('user'));
      this.resign.empId = user.employeeId;
      this.loading = true;
      this.getRequest(`/employee/resignation/?page=${this.page}&size=${this.size}&empId=${user.employeeId || ''}`).then(res => {
        this.resignList = res.data;
        this.total = res.total;
        this.loading = false;
      });
    },

    sizeChange(val) {
      this.size = val;
      this.getMyResignList();
    },
    currentChange(val) {
      this.page = val;
      this.getMyResignList();
    }
  }
};
</script>

<style>
.top-style {
  display: flex;
  justify-content: flex-start;
}
.content-style {
  margin-top: 10px;
}
.bottom-style {
  display: flex;
  justify-content: flex-end;
  margin-top: 10px;
}
</style>