<template>
  <div class="pane-container">
    <div class="toolbar">
      <el-button type="primary" @click="showAddDialog">
        <i class="el-icon-plus"></i>新增机构信息
      </el-button>
    </div>

    <el-table :data="list" border v-loading="loading" stripe style="width:100%">
      <el-table-column label="机构名称" prop="agencyName" width="220" />
      <el-table-column label="机构类型" prop="agencyType" width="130" />
      <el-table-column label="对接人" prop="contactPerson" width="110" />
      <el-table-column label="联系电话" prop="contactPhone" width="130" />
      <el-table-column label="供应类型" prop="supplyType" width="100" />
      <el-table-column label="输送岗位" prop="internPost" min-width="280">
        <template slot-scope="{ row }">
          {{ row.internPost ? row.internPost.replace(/,/g,'、') : '无' }}
        </template>
      </el-table-column>
      <el-table-column label="开户名" prop="bankAccountName" width="220" />
      <el-table-column label="开户银行" prop="bankName" width="200" />
      <el-table-column label="银行账号" prop="bankAccount" width="190" />
      <el-table-column label="计费方式" prop="serviceFeeMode" width="110" />
      <el-table-column label="管理费金额" prop="serviceFeeAmount" width="120" />
      <el-table-column label="结算周期" prop="settleCycle" width="110" />
      <el-table-column label="发票类型" prop="taxType" width="110" />
      <el-table-column label="机构地址" prop="agencyAddress" min-width="280" />
      <el-table-column label="备注" prop="remark" min-width="280" />

      <el-table-column fixed="right" width="140" label="操作">
        <template slot-scope="scope">
          <el-button @click="showEditDialog(scope.row)" style="padding:3px">编辑</el-button>
          <el-button @click="handleDelete(scope.row)" style="padding:3px" type="danger">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <div class="pagination-box">
      <el-pagination background @current-change="handlePageChange" @size-change="handleSizeChange"
                     layout="total, sizes, prev, pager, next, jumper"
                     :total="total" :page-sizes="[10,20,30]" :current-page="page" :page-size="size" />
    </div>

    <el-dialog :title="dialogTitle" center :visible.sync="dialogVisible" width="680px"
               :close-on-click-modal="false" top="8vh">
      <el-form :model="form" :rules="rules" ref="agencyForm" label-width="90px" class="form-grid">

        <!-- 基本信息 -->
        <div class="group-title">基本信息</div>
        <div class="form-row">
          <el-form-item label="机构名称" prop="agencyName" class="half">
            <el-input v-model="form.agencyName" placeholder="请输入机构名称"  style="width:180px"/>
          </el-form-item>
          <el-form-item label="机构类型" prop="agencyType" class="half">
            <el-select v-model="form.agencyType" placeholder="请选择机构类型" clearable  style="width:180px">
              <el-option label="职业院校" value="职业院校" />
              <el-option label="人力资源公司" value="人力资源公司" />
              <el-option label="劳务派遣公司" value="劳务派遣公司" />
              <el-option label="中介机构" value="中介机构" />
            </el-select>
          </el-form-item>
        </div>
        <div class="form-row">
          <el-form-item label="对接人" prop="contactPerson" class="half">
            <el-input v-model="form.contactPerson" placeholder="请输入对接人姓名"  style="width:180px"/>
          </el-form-item>
          <el-form-item label="联系电话" prop="contactPhone" class="half">
            <el-input v-model="form.contactPhone" placeholder="请输入联系电话"  style="width:180px"/>
          </el-form-item>
        </div>

        <!-- 供应类型独占一行，适配岗位紧跟在下方 -->
        <div class="form-row">
          <el-form-item label="供应类型" prop="supplyType" class="half">
            <el-select v-model="form.supplyType" placeholder="请选择供应类型" clearable  style="width:180px">
              <el-option label="实习生" value="实习生" />
              <el-option label="兼职" value="兼职" />
              <el-option label="寒暑假工" value="寒暑假工" />
              <el-option label="正式工" value="正式工" />
            </el-select>
          </el-form-item>
        </div>
        <el-divider>输送岗位</el-divider>
            <el-checkbox-group v-model="form.internPostList" class="post-checkbox-group" :key="isEdit ? 'edit' : 'add'">
              <el-checkbox label="服务员" />
              <el-checkbox label="迎宾" />
              <el-checkbox label="后厨" />
              <el-checkbox label="传菜" />
              <el-checkbox label="收银" />
              <el-checkbox label="保洁" />
            </el-checkbox-group>

        <!-- 账户与结算信息（可折叠） -->
        <el-collapse v-model="collapseActive" style="margin:10px 0;">
          <el-collapse-item title="账户与结算信息" name="bank">
            <div class="form-row">
              <el-form-item label="开户名" prop="bankAccountName" class="half">
                <el-input v-model="form.bankAccountName" placeholder="请输入开户名"  style="width:180px"/>
              </el-form-item>
              <el-form-item label="开户银行" prop="bankName" class="half">
                <el-input v-model="form.bankName" placeholder="请输入开户银行"  style="width:180px"/>
              </el-form-item>
            </div>
            <div class="form-row">
              <el-form-item label="银行账号" prop="bankAccount" class="half">
                <el-input v-model="form.bankAccount" placeholder="请输入银行账号"  style="width:180px"/>
              </el-form-item>
              <el-form-item label="计费方式" prop="serviceFeeMode" class="half">
                <el-select v-model="form.serviceFeeMode" placeholder="请选择计费方式" clearable  style="width:180px">
                  <el-option label="按人/月" value="按人/月" />
                  <el-option label="按天" value="按天" />
                  <el-option label="一次性" value="一次性" />
                </el-select>
              </el-form-item>
            </div>
            <div class="form-row">
              <el-form-item label="管理费" prop="form.serviceFeeAmount" class="half">
                <el-input-number
                    v-model="form.serviceFeeAmount"
                    controls-position="right"
                    :precision="2"
                    :step="1"
                    :min="0"
                    :max="999999.99"
                    placeholder="请输入管理费金额"
                    style="width: 180px"
                />
              </el-form-item>
              <el-form-item label="结算周期" prop="settleCycle" class="half">
                <el-select v-model="form.settleCycle" placeholder="请选择结算周期" clearable  style="width:180px">
                  <el-option label="月结" value="月结" />
                  <el-option label="周结" value="周结" />
                  <el-option label="半月结" value="半月结" />
                </el-select>
              </el-form-item>
            </div>
            <el-form-item label="发票类型" prop="taxType">
              <el-select v-model="form.taxType" placeholder="请选择发票类型" clearable style="width:180px">
                <el-option label="专票" value="专票" />
                <el-option label="普票" value="普票" />
                <el-option label="无票" value="无票" />
              </el-select>
            </el-form-item>
          </el-collapse-item>
        </el-collapse>

        <!-- 其他信息 -->
        <el-form-item label="机构地址" prop="agencyAddress">
          <el-input v-model="form.agencyAddress" type="textarea" rows="2" placeholder="请输入机构地址" />
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input v-model="form.remark" type="textarea" rows="2" placeholder="请输入备注信息" />
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取消</el-button>
    <el-button type="primary" @click="submitForm">保存</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "OrgAgency",
  data() {
    return {
      list: [],
      total: 0,
      page:1,
      size:10,
      loading:false,
      dialogVisible: false,
      dialogTitle: "新增机构信息",
      isEdit: false,
      editId: null,
      collapseActive: [],
      form: {
        agencyName:"",
        agencyType:"",
        contactPerson:"",
        contactPhone:"",
        supplyType:"",
        internPost:"",
        internPostList:[],
        bankAccountName:"",
        bankName:"",
        bankAccount:"",
        serviceFeeMode:"",
        serviceFeeAmount:"",
        settleCycle:"",
        taxType:"",
        agencyAddress:"",
        remark:""
      },

      rules: {
        agencyName: [{ required:true, message:"请输入机构全称", trigger:"blur" }],
        agencyType: [{ required:true, message:"请选择机构类型", trigger:"change" }],
        contactPerson: [{ required:true, message:"请输入对接人姓名", trigger:"blur" }],
        contactPhone: [
          { required:true, message:"请输入联系电话", trigger:"blur" },
          { pattern: /^1[3-9]\d{9}$/, message: "请输入正确的手机号", trigger: "blur" }
        ],
        supplyType: [{ required:true, message:"请选择供应类型", trigger:"change" }]
      }
    };
  },
  mounted() { this.getList(); },
  methods: {
    getList() {
      this.loading = true;
      this.getRequest(`/org/agency/?page=${this.page}&size=${this.size}`).then(res => {
        this.list = res.data;
        this.total = res.total;
        this.loading = false;
      });
    },
    showAddDialog() {
      this.isEdit=false;
      this.editId=null;
      this.dialogTitle = "新增机构信息";
      this.resetForm();
      this.collapseActive=[];
      this.dialogVisible=true;
      this.$nextTick(()=>this.$refs.agencyForm.clearValidate());
    },
    showEditDialog(row) {
      this.isEdit=true;
      this.editId=row.id;
      this.dialogTitle = "编辑机构信息";
      /*this.form = { ...row };*/
      // 先重置表单，保证 form 结构完整
      this.resetForm();

      // 再赋值
      this.form = { ...this.form, ...row };

      // 回显多选：把逗号分隔的字符串转成数组
      this.form.internPostList = row.internPost ? row.internPost.split(',') : [];
      this.collapseActive=["bank"];
      this.dialogVisible=true;
    },
    submitForm() {
      this.$refs.agencyForm.validate(valid=>{
        if(!valid) return;
        // 把多选数组转成逗号分隔的字符串，给后端
        this.form.internPost = this.form.internPostList.join(',');
        const api = this.isEdit ? this.postRequest("/org/agency/update",this.form)
            : this.postRequest("/org/agency/add",this.form);
        api.then(()=>{
          this.$message.success("保存成功");
          this.dialogVisible=false; this.getList();
        });
      });
    },
    handleDelete(row) {
      this.$confirm("确定删除该机构？","提示",{type:"warning"}).then(()=>{
        this.deleteRequest("/org/agency/"+row.id).then(()=>{
          this.$message.success("删除成功"); this.getList();
        });
      });
    },
    resetForm() {
      this.form = {
        agencyName:"",
        agencyType:"",
        contactPerson:"",
        contactPhone:"",
        supplyType:"",
        internPost:"",
        internPostList:[],
        bankAccountName:"",
        bankName:"",
        bankAccount:"",
        serviceFeeMode:"",
        serviceFeeAmount:"",
        settleCycle:"",
        taxType:"",
        agencyAddress:"",
        remark:""
      };
    },
    handlePageChange(val){this.page=val;this.getList();},
    handleSizeChange(val){this.size=val;this.page=1;this.getList();}
  }
};
</script>

<style scoped>
.pane-container { padding:20px; background:#fff; min-height:calc(100vh - 60px); }
.toolbar { margin-bottom:15px; }
.pagination-box { margin-top:15px; text-align:right; }
.dialog-footer { text-align:right; }

.group-title { font-size:14px; font-weight:600; margin:8px 0 12px; color:#333; }
.form-row { display:flex; justify-content:space-between; margin-bottom:12px; }
.form-row .half { width:48%; }

/* 收紧弹窗顶部留白，去掉基本信息上方的空隙 */
/deep/ .el-dialog__body {
  padding: 10px 20px 20px; /* 把顶部padding从默认的20px改成10px，收紧留白 */
}

/* 同时收紧表单内部的间距，让整体更紧凑 */
/deep/ .el-form-item {
  margin-bottom: 18px; /* 把默认的24px改成18px，缩小表单项之间的空隙 */
}

/* 适配岗位分割线间距优化 */
/deep/ .el-divider {
  margin: 12px 0; /* 缩小分割线上下的间距 */
}

/deep/ .el-divider__text {
  font-size: 14px;          /* 和表单标签字号一致 */
  color: #606266;           /* 和表单标签颜色一致（Element UI 标准灰） */
  font-weight: normal;      /* 去掉默认加粗，和其他标签统一 */
  background: #fff;         /* 确保背景色一致 */
}

/* 同时优化多选框的对齐和间距，保持美观 */
.post-checkbox-group {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  gap: 30px;
  padding: 8px 0;
}
.post-checkbox-group .el-checkbox {
  display: flex;
  align-items: center;
  margin: 0;
}
</style>