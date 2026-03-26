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
          <el-input v-model="promote.oldJobName" disabled style="width: 180px" />
        </el-form-item>

        <el-form-item label="晋升职位" prop="newPosId">
          <el-select v-model="promote.newPosId" placeholder="请选择晋升职位" style="width: 180px">
            <el-option v-for="item in positions" :key="item.id" :label="item.name" :value="item.id" />
          </el-select>
        </el-form-item>

        <el-form-item label="晋升职称" prop="newJobId">
          <el-select v-model="promote.newJobId" placeholder="请选择晋升职称" style="width: 180px">
            <el-option v-for="item in joblevels" :key="item.id" :label="item.name" :value="item.id" />
          </el-select>
        </el-form-item>

        <!-- 👇 这里新加：审批人选择框
        <el-form-item label="审批人" prop="approverId">
          <el-select v-model="promote.approverId" placeholder="请选择审批人" style="width: 180px">
            <el-option v-for="item in approvers" :key="item.id" :label="item.name" :value="item.id" />
          </el-select>
        </el-form-item>-->
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
                    :label="`${item.name} | ${item.department.name} | 工号:${item.workid}`"
                    :value="item.id"
            />
          </el-select>
        </el-form-item>

        <el-form-item label="晋升理由">
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
        <el-table-column label="原职位" prop="oldPosName" />
        <el-table-column label="申请晋升职位" prop="newPosName" />
        <el-table-column label="原职称" prop="oldJobName" />
        <el-table-column label="申请晋升职称" prop="newJobName" />
        <el-table-column label="审批人" prop="approverName" />
        <el-table-column label="申请理由" prop="reason" />
        <el-table-column label="申请时间" prop="applyTime" />
        <el-table-column label="审批状态">
          <template slot-scope="scope">
            <el-tag :type="scope.row.status === 0 ? 'warning' : scope.row.status === 1 ? 'success' : 'danger'">
              {{ scope.row.status === 0 ? '待审批' : scope.row.status === 1 ? '已通过' : '已拒绝' }}
            </el-tag>
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
          oldJobId: "",
          oldJobName: "",
          newJobId: "",
          newJobName: "",
          approverId: "", // 审批人ID
          approverName: "",
          reason: ""
        },
        rules: {
          newPosId: [{ required: true, message: "请选择晋升职位", trigger: "change" }],
          newJobId: [{ required: true, message: "请选择晋升职称", trigger: "change" }],
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
      // 加载所有基础数据
      loadBaseData() {
        this.getRequest("/employee/basic/positions").then(res => this.positions = res);
        this.getRequest("/employee/basic/joblevels").then(res => this.joblevels = res);
        this.getRequest("/employee/basic/employees").then(res => this.approvers = res); // 加载审批人
      },

      // 打开弹窗
      showApplyDialog() {
        this.promote = {};
        this.dialogVisible = true;
      },

      // 提交晋升申请
      submitPromote() {
        this.$refs.promoteForm.validate(valid => {
          if (valid) {
            this.postRequest("/promotion/apply", this.promote).then(res => {
              this.$notify.success("晋升申请提交成功！");
              this.dialogVisible = false;
              this.getMyPromotionList();
            });
          }
        });
      },

      // 获取自己的晋升记录
      getMyPromotionList() {
        /*this.loading = true;
        this.getRequest(`/promotion/myList?page=${this.page}&size=${this.size}`).then(res => {
          this.promoteList = res.data;
          this.total = res.total;
          this.loading = false;
        });*/
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