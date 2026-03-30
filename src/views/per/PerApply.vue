<template>
  <div>
    <!-- 顶部：只有 发起晋升申请 按钮 -->
    <div class="top-style">
      <el-button type="primary" icon="el-icon-plus" @click="showApplyDialog">
        发起晋升申请
      </el-button>
    </div>

    <!-- 晋升申请弹窗 -->
    <el-dialog title="员工晋升申请" center :visible.sync="dialogVisible" width="40%">
      <el-form :model="promote" :rules="rules" ref="promoteForm" label-width="110px">
        <el-form-item label="当前职位">
          <el-input v-model="promote.oldPosName" disabled style="width: 180px" />
        </el-form-item>

        <el-form-item label="当前职称">
          <el-input v-model="promote.oldJobLevelName" disabled style="width: 180px" />
        </el-form-item>

        <el-form-item label="晋升职位" prop="newPosId">
          <el-select v-model="promote.newPosId" placeholder="请选择晋升职位" style="width: 180px">
            <el-option v-for="item in positions" :key="item.id" :label="item.name" :value="item.id" />
          </el-select>
        </el-form-item>

        <el-form-item label="晋升职称" prop="newJobLevelId">
          <el-select v-model="promote.newJobLevelId" placeholder="请选择晋升职称" style="width: 180px">
            <el-option v-for="item in joblevels" :key="item.id" :label="item.name" :value="item.id" />
          </el-select>
        </el-form-item>

        <el-form-item label="审批人" prop="approverId">
          <el-select
                  v-model="promote.approverId"
                  placeholder="请搜索并选择审批人"
                  style="width: 280px"
                  filterable
                  clearable
          >
            <!-- 这里完全匹配你返回的字段：name + 部门 + 工号 -->
            <el-option
                    v-for="item in approvers"
                    :key="item.id"
                    :label="`${item.name} | ${item.department.name} | ${item.position.name}`"
                    :value="item.id"
            />
          </el-select>
        </el-form-item>

        <el-form-item label="晋升理由" prop="reason">
          <el-input
                  type="textarea"
                  v-model="promote.reason"
                  maxlength="100"
                  show-word-limit
                  style="width: 350px"
                  rows="3"
          />
        </el-form-item>
      </el-form>

      <div slot="footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submitPromote">提交申请</el-button>
      </div>
    </el-dialog>

    <!-- ========== 新增：审批意见弹窗 ========== -->
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
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="approvalDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submitApproval">确认提交</el-button>
      </div>
    </el-dialog>
    <!-- 晋升记录列表 -->
    <div class="content-style" style="margin-top:10px;">
      <el-table
              :data="promoteList"
              border
              stripe
              max-height="680"
              v-loading="loading"
              style="width:100%"
      >
        <el-table-column label="申请人" prop="applyName"  width="100"/>
        <el-table-column label="原职位" prop="oldPosName"  width="100"/>
        <el-table-column label="申请晋升职位" prop="newPosName"  width="120"/>
        <el-table-column label="原职称" prop="oldJobName"  width="100"/>
        <el-table-column label="申请晋升职称" prop="newJobName"  width="120"/>
        <el-table-column label="审批人" prop="approverName" width="100"/>
        <el-table-column label="申请理由" width="210" prop="reason"  />
        <el-table-column label="申请时间" prop="applyTime" width="160"/>
        <el-table-column label="审核时间" width="160">
          <template slot-scope="scope">
            {{ scope.row.approveTime || '-' }}
          </template>
        </el-table-column>
        <el-table-column label="审核意见" prop="approveRemark" width="210">
          <template slot-scope="scope">
            {{ scope.row.approveRemark || '-' }}
          </template>
        </el-table-column>
        <el-table-column label="审批状态"  width="100">
          <template slot-scope="scope">
            <el-tag :type="scope.row.status === 0 ? 'warning' : scope.row.status === 1 ? 'success' : 'danger'">
              {{ scope.row.status === 0 ? '待审批' : scope.row.status === 1 ? '已通过' : '已拒绝' }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" fixed="right" width="120">
          <template slot-scope="scope">
            <!-- 🔹 我是审批人（approverId === 当前登录人ID）：显示 通过 / 拒绝 -->
            <el-button
                    v-if="scope.row.approverId === promote.empId && scope.row.status === 0"
                    type="success"
                    style="padding: 3px"
                    @click="handlePreApprove(scope.row, 1)"
            >
              通过
            </el-button>
            <el-button
                    v-if="scope.row.approverId === promote.empId && scope.row.status === 0"
                    type="danger"
                    style="padding: 3px"
                    @click="handlePreApprove(scope.row, 2)"
            >
              拒绝
            </el-button>
            <!-- 🔹 我是申请人（自己的申请）且待审批：显示 删除 -->
            <el-button
                    v-else-if="scope.row.empId === promote.empId && scope.row.status === 0"
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
    name: "PromotionApply",
    data() {
      return {
        promoteList: [],
        loading: false,
        dialogVisible: false,
        total: 0,
        page: 1,
        size: 10,
        positions: [],
        joblevels: [],
        approvers: [], // 审批人列表
        promote: {
          id: "",
          empId: "",
          oldPosId: "",
          oldPosName: "",
          newPosId: "",
          newPosName: "",
          oldJobLevelId: "",
          oldJobLevelName: "",
          newJobLevelId: "",
          newJobLevelName: "",
          approverId: "", // 审批人ID
          approverName: "",
          reason: ""
        },
        approvalDialogVisible: false, // 审批弹窗
        approvalForm: { // 审批表单
          id: '',
          empId:'',
          status: '',
          newPosId: "",
          newJobLevelId: "",
          remark: ''
        },
        rules: {
          newPosId: [{ required: true, message: "请选择晋升职位", trigger: "change" }],
          newJobLevelId: [{ required: true, message: "请选择晋升职称", trigger: "change" }],
          approverId: [{ required: true, message: "请选择审批人", trigger: "change" }],
          reason: [{ required: true, message: "请填写晋升理由", trigger: "blur" }]
        }
      };
    },
    mounted() {
      this.loadBaseData();
      this.getMyPromotionList();
    },
    methods: {
      // 1. 打开审批弹窗（准备通过/拒绝）
      handlePreApprove(row, status) {
        this.approvalForm.id = row.id; // 赋值申请ID
        this.approvalForm.empId = row.empId;
        this.approvalForm.newPosId = row.newPosId;
        this.approvalForm.newJobLevelId = row.newJobLevelId;
        this.approvalForm.status = status; // 1=通过，2=拒绝
        this.approvalForm.remark = ''; // 清空上次内容
        this.approvalDialogVisible = true; // 打开弹窗
      },

// 2. 最终提交审批
      submitApproval() {
        if (!this.approvalForm.remark) {
          this.$message.warning('请填写审批意见！');
          return;
        }

        // 调用接口
        this.postRequest(`/employee/promotion/update/`, {
          id:this.approvalForm.id,
          empId:this.approvalForm.empId,
          status: this.approvalForm.status,
          approveRemark: this.approvalForm.remark, // 把原因传给后端
          newPosId: this.approvalForm.newPosId,
          newJobLevelId: this.approvalForm.newJobLevelId,
        }).then(res => {
          this.$message.success('审核成功！');
          this.approvalDialogVisible = false;
          this.getMyPromotionList(); // 刷新列表
        });
      },

      handleDelete(row) {
        this.$confirm('确定要删除这条待审批的晋升申请吗？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.deleteRequest("/employee/promotion/"+row.id).then(res => {
            this.$message.success('删除成功！');
            this.getMyPromotionList(); // 刷新列表
          });
        }).catch(() => {});
      },
      // 加载所有基础数据
      loadBaseData() {
        this.getRequest("/employee/basic/positions").then(res => this.positions = res);
        this.getRequest("/employee/basic/joblevels").then(res => this.joblevels = res);
        this.getRequest("/employee/basic/employees").then(res => this.approvers = res); // 加载审批人
      },

      // 打开弹窗
      showApplyDialog() {
        const hasPendingApply = this.promoteList.some(item =>
                item.empId === this.promote.empId && item.status === 0
        );

        // 如果有待审批，直接提示并返回，不打开弹窗
        if (hasPendingApply) {
          this.$message.warning('你已有待审批的晋升申请，不可重复提交！');
          return;
        }

        this.getRequest('/employee/basic/findOne?id='+this.promote.empId).then(resp => {
          if (resp) {
            let emp = resp;
            // 2. 自动带入当前职位和职称
            this.promote.oldPosId = emp.position.id;
            this.promote.oldPosName = emp.position.name;
            this.promote.oldJobLevelId = emp.jobLevel.id;
            this.promote.oldJobLevelName = emp.jobLevel.name;

          }

        });

        this.dialogVisible = true;
        // ✅ 关键：打开弹窗后，立即清除校验提示（必须放在这里！）
        this.$nextTick(() => {
          if (this.$refs.promoteForm) {
            this.$refs.promoteForm.clearValidate();
          }
        });
      },

      // 提交晋升申请
      submitPromote() {
        this.$refs.promoteForm.validate(valid => {
          if (valid) {
            // =========== 业务判断条件 ===========


            const posChanged = this.promote.newPosId !== this.promote.oldPosId;
            const jobChanged = this.promote.newJobLevelId !== this.promote.oldJobLevelId;

            // 2. 如果两个都没变化 → 拦截
            if (!posChanged && !jobChanged) {
              this.$message.error("职位和职级至少需要修改一项！");
              return;
            }

            // 3. 必须填理由
            if (!this.promote.reason || this.promote.reason.trim() === '') {
              this.$message.error('请填写晋升理由！');
              return;
            }

            // 4. 必须选审批人
            if (!this.promote.approverId) {
              this.$message.error('请选择审批人！');
              return;
            }

            this.postRequest("/employee/promotion/add", this.promote).then(res => {
              this.$notify.success("晋升申请提交成功！");
              this.dialogVisible = false;
              this.promote = {
                oldPosId: "",
                oldPosName: "",
                newPosId: "",
                newPosName: "",
                oldJobLevelId: "",
                oldJobLevelName: "",
                newJobLevelId: "",
                newJobName: "",
                approverId: "",
                approverName: "",
                reason: ""
              };

              this.getMyPromotionList();
            });
          }
        });
      },

      // 获取自己的晋升记录
      getMyPromotionList() {
        // 取当前登录用户
        let user = JSON.parse(sessionStorage.getItem('user'));
        this.promote.empId = user.employeeId;
        this.loading = true;
        this.getRequest(`/employee/promotion/?page=${this.page}&size=${this.size}&empId=`+(user.employeeId || '')).then(res => {
          this.promoteList = res.data;
          this.total = res.total;
          this.loading = false;
        });
      },

      sizeChange(val) {
        this.size = val;
        this.getMyPromotionList();
      },
      currentChange(val) {
        this.page = val;
        this.getMyPromotionList();
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