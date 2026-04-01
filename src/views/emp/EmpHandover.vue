<template>
  <div class="pane-container">

    <!-- 工具栏 -->
    <div class="toolbar">
      <!-- 这里改成：新增交接记录 ✅ -->
      <el-button type="primary" @click="openDialog">
        <i class="el-icon-plus"></i>新增交接记录
      </el-button>
    </div>

    <!-- 列表 -->
    <el-table :data="list" border v-loading="loading" stripe>
      <el-table-column label="交接人" prop="handoverBy" width="120" />
      <el-table-column label="接收人" prop="receiveBy" width="120" />
      <el-table-column label="交接时间" prop="handoverTime" width="180" />
      <el-table-column label="交接物品" min-width="300">
        <template slot-scope="{ row }">
          {{ row.itemNames.replace(/,/g, '、') }}
        </template>
      </el-table-column>
      <el-table-column label="备注" prop="remark" min-width="200" />
    </el-table>


    <div class="bottom-style">
      <el-pagination
          style="margin-top:10px"
          background
          @current-change="getList"
          @size-change="getList"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          :page-sizes="[10, 20, 30]"
      />
    </div>

    <!-- 弹框标题也改：新增交接记录 ✅ -->
    <el-dialog
        title="新增交接记录"
        :visible.sync="dialogVisible"
        width="680px"
        @close="resetForm"
    >
      <el-form :model="form" :rules="rules" ref="formRef" label-width="80px">
        <el-form-item label="交接人" prop="handoverBy">
          <el-select
              v-model="form.handoverBy"
              placeholder="请搜索并选择交接人"
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
        <el-form-item label="接收人" prop="receiveBy">
          <el-select
              v-model="form.receiveBy"
              placeholder="请搜索并选择接收人"
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

        <el-form-item label="交接时间" prop="handoverTime">
          <el-date-picker
              v-model="form.handoverTime"
              type="datetime"
              style="width:100%"
              value-format="yyyy-MM-dd HH:mm:ss"
          />
        </el-form-item>
        <el-form-item label="备注">
          <el-input v-model="form.remark" type="textarea" rows="3" style="width:100%" />
        </el-form-item>

        <el-divider>交接物品</el-divider>

        <el-checkbox-group v-model="form.itemNames" class="checkbox-group">
          <el-checkbox label="大门钥匙" />
          <el-checkbox label="收银台钥匙" />
          <el-checkbox label="仓库钥匙" />
          <el-checkbox label="保险柜钥匙" />
          <el-checkbox label="收银机" />
          <el-checkbox label="POS机" />
          <el-checkbox label="备用金" />
          <el-checkbox label="营业款" />
          <el-checkbox label="发票本" />
          <el-checkbox label="对讲机" />
          <el-checkbox label="空调遥控器" />
          <el-checkbox label="灭火器" />
          <el-checkbox label="水电气总阀" />
          <el-checkbox label="监控设备" />
        </el-checkbox-group>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submit">确认保存</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "HandoverRecord",
  data() {
    return {
      list: [],
      total: 0,
      pageNum: 1,
      pageSize: 10,
      loading: false,
      dialogVisible: false,
      formRef: null,
      form: {
        handoverBy: "",
        receiveBy: "",
        handoverTime: "",
        remark: "",
        itemNames: []
      },
      rules: {
        handoverBy: [{ required: true, message: "请选择交接人", trigger: "blur" }],
        receiveBy: [{ required: true, message: "请选择接收人", trigger: "blur" }],
        handoverTime: [{ required: true, message: "请选择交接时间", trigger: "change" }]
      }
    };
  },
  created() {
    this.getList();
  },
  methods: {
    async getList() {
      this.loading = true;
      const res = await this.$http.get("/handover/list", {
        params: { pageNum: this.pageNum, pageSize: this.pageSize }
      });
      this.list = res.data.records || res.data.list;
      this.total = res.data.total || 0;
      this.loading = false;
    },
    openDialog() {
      this.dialogVisible = true;
    },
    resetForm() {
      this.$refs.formRef && this.$refs.formRef.resetFields();
      this.form.itemNames = [];
    },
    async submit() {
      await this.$refs.formRef.validate();
      if (this.form.itemNames.length === 0) {
        this.$message.warning("请至少选择一项交接物品");
        return;
      }

      const params = {
        ...this.form,
        itemNames: this.form.itemNames.join(",")
      };

      // 直接保存，无审批 ✅
      await this.$http.post("/handover/add", params);
      this.$message.success("保存成功");
      this.dialogVisible = false;
      this.getList();
    }
  }
};
</script>

<style scoped>
.pane-container {
  padding: 20px;
  background: #fff;
}
.breadcrumb {
  font-size: 14px;
  color: #666;
  margin-bottom: 16px;
}
.breadcrumb span {
  margin: 0 4px;
}
.toolbar {
  margin-bottom: 15px;
}
.checkbox-group {
  display: flex;
  flex-wrap: wrap;
  gap: 18px 25px;
}
.dialog-footer {
  text-align: right;
}
</style>