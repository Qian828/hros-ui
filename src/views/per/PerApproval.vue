<template>
  <div>
    <div class="top-style">
      <el-input placeholder="搜索员工" v-model="keyword" style="width: 350px"
                prefix-icon="el-icon-search" clearable @keydown.enter.native="initData"></el-input>
    </div>

    <div class="content-style" style="margin-top:10px;">
      <el-table :data="list" border stripe max-height="680" v-loading="loading">
        <el-table-column label="姓名" prop="name"></el-table-column>
        <el-table-column label="申请职位" prop="newPosName"></el-table-column>
        <el-table-column label="申请职称" prop="newJobName"></el-table-column>
        <el-table-column label="申请人" prop="applyUserName"></el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-tag :type="scope.row.status ===0 ? 'warning' : scope.row.status===1 ? 'success':'danger'">
              {{ scope.row.status ===0 ? '待审批' : scope.row.status===1 ? '已通过':'已拒绝' }}
            </el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="200">
          <template slot-scope="scope">
            <el-button type="success" size="mini" @click="approve(scope.row.id,1)" :disabled="scope.row.status!==0">同意</el-button>
            <el-button type="danger" size="mini" @click="approve(scope.row.id,2)" :disabled="scope.row.status!==0">拒绝</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      keyword: '',
      list: [],
      loading: false
    }
  },
  mounted() {
    this.initData();
  },
  methods: {
    initData() {
      this.loading = true;
      this.getRequest("/promotion/approval?keyword=" + this.keyword).then(res => {
        this.list = res.data;
        this.loading = false;
      })
    },
    approve(id, status) {
      this.postRequest("/promotion/approval", { id, status }).then(res => {
        this.$notify.success(status === 1 ? "已同意" : "已拒绝");
        this.initData();
      })
    }
  }
}
</script>

<style>
.top-style { display: flex; justify-content: space-between; }
</style>