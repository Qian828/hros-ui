<template>
  <div>
    <div class="top-style">
      <div style="display: flex;justify-content: flex-start;">
        <el-input placeholder="请输入姓名进行搜索，可以直接回车搜索..."
                  prefix-icon="el-icon-search"
                  clearable
                  @clear="initList"
                  style="width: 350px;margin-right: 10px"
                  v-model="keyword"
                  @keydown.enter.native="initList"
                  :disabled="showAdvanceSearchView"></el-input>
        <el-button icon="el-icon-search"
                   type="primary"
                   @click="initList"
                   :disabled="showAdvanceSearchView">
          搜索
        </el-button>
      </div>

      <div style="display: flex;justify-content: flex-end">
        <el-button type="primary"
                   icon="el-icon-plus"
                   @click="showAddEmpView">
          添加黑名单人员
        </el-button>
        <!--add 弹窗-->
        <el-dialog :title="title"
                   :visible.sync="dialogVisible"
                   width="40%"
                   append-document>
          <div>
            <!-- 关键修复1：将ref从empForm改为blackForm，避免命名冲突 -->
            <el-form :model="blackForm"
                     :rules="rules"
                     ref="blackForm"
                     label-width="120px">

              <!-- 隐藏域：主键ID (后端维护) -->
              <el-input v-model="blackForm.id" type="hidden"></el-input>

              <el-row :gutter="20">
                <el-col :span="12">
                  <el-form-item label="姓名:" prop="name">
                    <el-input prefix-icon="el-icon-edit"
                              v-model="blackForm.name"
                              placeholder="请输入姓名"></el-input>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="身份证号:" prop="idCard">
                    <el-input prefix-icon="el-icon-edit"
                              v-model="blackForm.idcard"
                              placeholder="请输入身份证号码"></el-input>
                  </el-form-item>
                </el-col>
              </el-row>

              <el-row :gutter="20">
                <el-col :span="12">
                  <el-form-item label="联系电话:" prop="phone">
                    <el-input prefix-icon="el-icon-phone"
                              v-model="blackForm.phone"
                              placeholder="请输入联系电话"></el-input>
                  </el-form-item>
                </el-col>
                <el-col :span="12">
                  <el-form-item label="所属部门:" prop="sourcedepartment">
                    <el-popover placement="right"
                                title="请选择部门"
                                width="200"
                                trigger="manual"
                                v-model="popVisible">
                      <el-tree default-expand-all
                               :data="allDeps"
                               :props="defaultProps"
                               @node-click="handleNodeClick"></el-tree>
                      <div slot="reference"
                           style="width: 100%;display: inline-flex;font-size: 12px;border: 1px solid #dedede;height: 38px;border-radius: 5px;cursor: pointer;align-items: center;box-sizing: border-box;padding: 0 10px;"
                           @click="showDepView1()">{{ inputDepName }}
                      </div>
                    </el-popover>
                  </el-form-item>
                </el-col>
              </el-row>

              <!-- 关键新增：黑名单原因 -->
              <el-row :gutter="20">
                <el-col :span="24">
                  <el-form-item label="黑名单原因:" prop="reason">
                    <el-input type="textarea"
                              rows="3"
                              prefix-icon="el-icon-warning"
                              v-model="blackForm.reason"
                              placeholder="请输入加入黑名单的原因（如：旷工、违规操作等）"></el-input>
                  </el-form-item>
                </el-col>
              </el-row>

              <el-row :gutter="20">
                <el-col :span="12">
                  <el-form-item label="加入时间:" prop="addTime">
                    <el-date-picker v-model="blackForm.addTime"
                                    type="datetime"
                                    value-format="yyyy-MM-dd HH:mm:ss"
                                    style="width: 100%;"
                                    placeholder="选择加入时间">
                    </el-date-picker>
                  </el-form-item>
                </el-col>

              </el-row>

              <el-row :gutter="20">
                <el-col :span="24">
                  <el-form-item label="备注:" prop="remark">
                    <el-input type="textarea"
                              rows="3"
                              prefix-icon="el-icon-note"
                              v-model="blackForm.remark"
                              placeholder="请输入备注信息"></el-input>
                  </el-form-item>
                </el-col>
              </el-row>
            </el-form>
          </div>
          <span slot="footer"
                class="dialog-footer">
            <el-button @click="quxiao">取 消</el-button>
            <el-button type="primary"
                       @click="doSubmit">确 定</el-button>
          </span>
        </el-dialog>
      </div>
    </div>

    <div class="content-style">
      <el-scrollbar style="height: 100%">
        <div>
          <el-table :data="blackList"
                    stripe
                    border
                    max-height="685"
                    v-loading.fullscreen.lock="loading"
                    element-loading-spinner="fa fa-spinner fa-pulse fa-3x fa-fw"
                    style="width: 100%;height: 695px"
                    @selection-change="handleSelectionChange">
            <el-table-column type="selection" width="55"/>

            <el-table-column fixed prop="name" label="姓名" width="100"/>
            <el-table-column prop="idcard" label="身份证号" width="180"/>
            <el-table-column prop="phone" label="电话" width="130"/>

            <!-- 关键修复：表格中加入原因列 -->
            <el-table-column prop="reason" label="黑名单原因" min-width="200" show-overflow-tooltip/>

            <el-table-column prop="departName" label="所属部门" width="120"/>
            <el-table-column prop="addtime" label="加入时间" width="160"/>
            <el-table-column prop="remark" label="备注" width="200" show-overflow-tooltip/>
            <!-- 状态列优化 -->
            <el-table-column prop="status" label="状态" width="100" align="center">
              <template slot-scope="scope">
                <el-tag :type="scope.row.status === 1 ? 'success' : 'danger'">
                  {{ scope.row.status === 1 ? '有效' : '已失效' }}
                </el-tag>
              </template>
            </el-table-column>
            <el-table-column fixed="right"
                             width="150"
                             label="操作">
              <template slot-scope="scope">
                <el-button
                    @click="showEditView(scope.row)"
                    style="padding: 3px"
                    :disabled="scope.row.status === 0">
                  编辑
                </el-button>
                <el-button
                    @click="deleteItem(scope.row)"
                    style="padding: 3px"
                    type="danger"
                    :disabled="scope.row.status === 0">

                  移除黑名单
                </el-button>
              </template>
            </el-table-column>

          </el-table>
        </div>
      </el-scrollbar>
    </div>

    <div class="bottom-style">
      <el-button type="danger"
                 style="margin-top: 10px"
                 :disabled="multipleSelection.length == 0"
                 @click="deleteMany">批量移除黑名单
        </el-button>
        <el-pagination style="margin-top: 10px"
                       background
                       @current-change="currentChange"
                       @size-change="sizeChange"
                       layout="sizes, prev, pager, next, jumper, ->, total"
                       :total="total"
                       :page-sizes="[10, 20, 50, 100]">
        </el-pagination>
    </div>
  </div>
</template>

<script>
export default {
  name: "BlackList",
  data() {
    return {
      multipleSelection: [],
      showAdvanceSearchView: false,
      allDeps: [],
      blackList: [], // 重命名：emps -> blackList
      loading: false,
      popVisible: false,
      dialogVisible: false,
      total: 0,
      page: 1,
      size: 10,
      keyword: '',
      title: '',
      inputDepName: '',
      // 关键修复：表单数据结构与数据库字段一一对应
      blackForm: {
        id: '',
        name: '',
        idcard: '',
        phone: '',
        sourcedepartment: '', // 对应数据库 sourceDepartment
        reason: '',           // 新增：黑名单原因
        addTime: '',
        operator: '',         // 操作员 (通常由前端传当前登录用户)
        remark: '',
        status: 1             // 默认有效
      },

      defaultProps: {
        children: 'children',
        label: 'name'
      },

      // 校验规则更新
      rules: {
        name: [{required: true, message: '请输入姓名', trigger: 'blur'}],
        idcard: [
          {required: true, message: '请输入身份证号', trigger: 'blur'},
          {pattern: /(^\d{18}$)|(^\d{17}(\d|X|x)$)/, message: '身份证格式错误', trigger: 'blur'}
        ],
        phone: [{required: true, message: '请输入电话', trigger: 'blur'}],
        sourcedepartment: [{required: true, message: '请选择部门', trigger: 'blur'}],
        reason: [{required: true, message: '请输入黑名单原因', trigger: 'blur'}], // 必填
        addTime: [{required: true, message: '请选择加入时间', trigger: 'change'}],
      }
    }
  },
  mounted() {
    this.initList();
    this.initData();
  },
  methods: {
    // 初始化数据列表
    initList() {
      this.loading = true;
      let url = `/employee/black/?page=${this.page}&size=${this.size}`;
      if (this.keyword) {
        url += `&name=${this.keyword}`;
      }

      this.getRequest(url).then(resp => {
        this.loading = false;
        if (resp && resp.data) {
          this.blackList = resp.data.records || resp.data; // 适配分页结构
          this.total = resp.data.total || 0;
        }
      }).catch(() => this.loading = false);
    },

    initData() {
      // 部门数据加载
      if (!window.sessionStorage.getItem("deps")) {
        this.getRequest('/employee/basic/deps').then(resp => {
          if (resp) {
            this.allDeps = resp;
            window.sessionStorage.setItem("deps", JSON.stringify(resp));
          }
        })
      } else {
        this.allDeps = JSON.parse(window.sessionStorage.getItem("deps"));
      }
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    handleNodeClick(data) {
      this.blackForm.sourcedepartment = data.id;
      this.inputDepName = data.name;
      // this.blackForm.departmentId = data.id; // 如果后端需要departmentId
      this.popVisible = false;
    },

    showDepView1() {
      this.popVisible = !this.popVisible;
    },

    // 新增/编辑弹窗
    showAddEmpView() {
      this.blackForm = { // 重置表单
        id: '',
        name: '',
        idcard: '',
        phone: '',
        sourcedepartment: '',
        reason: '',
        addTime: new Date(), // 默认当前时间
        operator: JSON.parse(window.sessionStorage.getItem("user")).name || '系统', // 获取当前操作员
        remark: '',
        status: 1
      };
      this.title = '添加黑名单人员';
      this.dialogVisible = true;
      this.$nextTick(() => this.$refs.blackForm.clearValidate());
    },

    showEditView(row) {
      // 深拷贝数据，避免污染原数据
      this.blackForm = JSON.parse(JSON.stringify(row));
      // 手动赋值部门名称，用于输入框回显
      this.inputDepName = row.departName;
      // 时间字符串转Date对象，解决date-picker回显问题
      if (row.addtime) {
        this.blackForm.addTime = new Date(row.addtime);
      }
      //this.blackForm = { ...row }; // 深拷贝数据
      this.title = '编辑黑名单人员';
      this.dialogVisible = true;
      this.$nextTick(() => this.$refs.blackForm.clearValidate());
    },

    doSubmit() {
      if (this.blackForm.id) {
        this.$refs['blackForm'].validate(valid => {
          if (valid) {
            this.$notify.success({
              title: '修改讯息',
              message: ' 修 改 员 工 中...',
              showClose: false,
              offset: 100,
              duration: 1500,
              customClass: 'fontclass'
            });
            this.putRequest("/employee/black/update", this.blackForm).then(resp => {
              if (resp) {
                this.dialogVisible = false;
                this.initList();
              }
            })
          }
        });
      } else {
        this.$refs['blackForm'].validate(valid => {
          if (valid) {
            this.$notify.success({
              title: '添加讯息',
              message: '添 加 员 工 中...',
              showClose: false,
              offset: 100,
              duration: 1500,
              customClass: 'fontclass'
            });
            this.postRequest("/employee/black/add", this.blackForm).then(resp => {
              if (resp) {
                this.dialogVisible = false;
                this.initList();
              }
            })
          }
        });
      }
    },

    // 提交表单 (整合新增和编辑)
   /* doSubmit() {
      this.$refs.blackForm.validate((valid) => {
        if (valid) {
          const url = this.blackForm.id ? '/employee/black' : '/employee/black';
          const method = this.blackForm.id ? 'put' : 'post';

       /!*   this[`${method}Request`](sslocal://flow/file_open?url=url%2C+this.blackForm&flow_extra=eyJsaW5rX3R5cGUiOiJjb2RlX2ludGVycHJldGVyIn0=).then(resp => {
          if (resp) {
            this.$message.success(`操作成功！`);
            this.dialogVisible = false;
            this.initList();
          }*!/
        }
      });
    },*/

deleteItem(row) {
  this.$confirm('确认移除该记录吗？', '提示', { type: 'warning' }).then(() => {
    this.deleteRequest(`/employee/black/${row.id}`).then(resp => {
      this.$message.success('移除成功！');
      this.initList();
    });
  });
},

deleteMany() {
  const hasInvalid = this.multipleSelection.some(item => item.status === 0);

  if (hasInvalid) {
    this.$message.error('选中项包含已失效数据，请重新选择！');
    this.multipleSelection = []; // 清空勾选
    return;
  }

  const ids = this.multipleSelection.map(item => item.id).join(',');
  this.$confirm(`确认移除选中的 ${this.multipleSelection.length} 条记录？`, '提示', { type: 'warning' })
      .then(() => {
        this.deleteRequest(`/employee/black/many/?ids=${ids}`).then(resp => {
          this.$message.success('批量移除成功！');
          this.initList();
        });
      });
},

// 分页事件
sizeChange(val) {
  this.size = val;
  this.initList();
},
currentChange(val) {
  this.page = val;
  this.initList();
},

quxiao() {
  this.dialogVisible = false;
}
}
}
</script>

<style scoped>
.top-style {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}
.content-style {
  margin-top: 10px;
  min-height: 400px;
}
.bottom-style {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 10px;
}
</style>