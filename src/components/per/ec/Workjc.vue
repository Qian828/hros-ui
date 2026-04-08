<template>
  <div>
    <!-- 顶部搜索+统计栏 -->
    <div class="top-style">
      <div style="display: flex;align-items: center;justify-content: space-between;">
        <!-- 搜索区 -->
        <div style="display: flex;align-items: center;">
          <el-input
              placeholder="请输入员工名进行搜索，可以直接回车搜索..."
              prefix-icon="el-icon-search"
              clearable
              @clear="initEmpAttendance"
              style="width: 350px;margin-right: 10px"
              v-model="keyword"
              @keydown.enter.native="initEmpAttendance"
          />
          <el-button
              icon="el-icon-search"
              type="primary"
              @click="initEmpAttendance"
          >
            搜索
          </el-button>
        </div>

        <!-- 考勤统计区（替换原来的死按钮） -->
        <div class="stat-box">
          <span class="stat-item">本月排班：{{ scheduleCount }}天</span>
          <span class="stat-item sign">已签到：{{ signCount }}天</span>
          <span class="stat-item absent">缺勤：{{ absentCount }}天</span>
        </div>
      </div>
    </div>

    <!-- 日历主体 -->
    <el-calendar v-model="currentMonth" :first-day-of-week="7">
      <template slot="dateCell" slot-scope="{ data }">
        <div class="calendar-cell" :class="getCellClass(data.day)">
                  <!-- 日期号 -->
                  <div class="day-text">
                    {{ data.day.split('-').slice(2).join('-') }}
                    <div v-if="getShift(data.day)"  class="shift-tag" :class="getShiftClass(data.day)">
                      {{ getShift(data.day) }}班
                    </div>
                  </div>

                  <!-- 1. 班次标签（已排班才显示） -->

          <!-- 2. 已签到标记 -->
          <div v-if="isSigned(data.day)" class="signed-tag">
            已签到
          </div>

          <!-- 3. 签到按钮：仅 已排班+未签到+非未来日期 显示 -->
          <el-button
              v-if="!isFuture(data.day) && getShift(data.day) && !isSigned(data.day)"
              type="primary"
            size="medium"
            icon="el-icon-check"
            class="sign-btn-improved"
              @click="handleSign(data)"
          >
            签到
          </el-button>
          <!-- 4. 未排班提示：历史日期+未排班 -->
          <div
              v-if="!isFuture(data.day) && !getShift(data.day) && !isSigned(data.day)"
              class="no-schedule"
          >
            未排班
          </div>

          <!-- 5. 未来日期提示 -->
          <div v-if="isFuture(data.day)" class="future-tag">
            未到时间
          </div>
        </div>
      </template>
    </el-calendar>
  </div>
</template>

<style scoped>
/* 顶部统计栏 */
.stat-box {
  display: flex;
  gap: 20px;
}
.sign-btn-improved {
  /* 强制覆盖样式，确保按钮够大 */
  font-size: 13px !important;
  padding: 8px 8px !important; /* 上下内边距变大，按钮更饱满 */
  height: auto !important; /* 取消固定高度，适应内边距 */
  font-weight: 500 !important;
}
.stat-item {
  padding: 4px 12px;
  border-radius: 4px;
  background: #f5f7fa;
  color: #606266;
  font-size: 14px;
}
.stat-item.sign {
  background: #f0f9ff;
  color: #1890ff;
  border-radius: 4px;
  font-weight: bold;
  font-size: 14px;
}
.stat-item.absent {
  background: #fff2f0;
  border-radius: 4px;
  color: #ff4d4f;
  font-weight: bold;
  font-size: 14px;
}

/* 日历单元格 */
.calendar-cell {
  min-height: 60px;
  padding: 4px;
}
.day-text {
  font-weight: bold;
  font-size: 14px;
  margin-bottom: 4px;
}
/* 今天高亮 */
.today-cell {
  background: #e6f7ff;
  border-radius: 4px;
}
/* 未来日期弱化 */
.future-cell {
  opacity: 0.5;
}

/* 班次标签 */
.shift-tag {
  margin-left: auto;
  font-size: 12px;
  padding: 2px 4px;
  border-radius: 3px;
  margin-bottom: 2px;
  display: inline-block;
  border-radius: 3px;
}
.shift-tag.morning {
  background: #f6ffed;
  color: #52c41a;
}
.shift-tag.afternoon {
  background: #fff7e6;
  color: #fa8c16;
}
.shift-tag.evening {
  background: #e0f7fa;
  color: #08979c;
}

/* 已签到标签 */
.signed-tag {
  color: #f5222d;
  font-size: 12px;
  font-weight: bold;
  margin-bottom: 2px;
}
/* 未排班/未来标签 */
.no-schedule, .future-tag {
  color: #999;
  font-size: 12px;
}
/* 背景色区分早中晚 */
.calendar-cell.morning {
  background: #f6ffed !important;
}
.calendar-cell.afternoon {
  background: #fff7e6 !important;
}
.calendar-cell.evening {
  background: #e0f7fa !important;
}
</style>

<script>
export default {
  name: "AttendanceManage",
  data() {
    return {
      keyword: "",
      currentMonth: new Date(), // 当前日历月份
      emps: [],
      total: 0,
      page: 1,
      size: 12,
      // 考勤核心数据
      scheduleMap: {}, // 排班映射 {日期: 班次}
      signDates: [], // 已签到日期数组
      // 统计数据
      scheduleCount: 0,
      signCount: 0,
      absentCount: 0,
    };
  },
  mounted() {
    this.initEmpAttendance();
  },
  methods: {
    // ======================== 日期基础判断 ========================
    // 判断是否是未来日期
    isFuture(day) {
      if (!day) return false;

      // 只比较 年月日，彻底解决时区/时间问题
      const today = new Date();
      const todayStr = today.toISOString().split('T')[0]; // 2026-04-08

      // 未来 = 日期 > 今天
      return day > todayStr;
    },
    // 判断是否是今天
    isToday(day) {
      const today = new Date().toISOString().split('T')[0];
      return day === today;
    },
    // 获取单元格样式类
    getCellClass(day) {
      let className = "";
      if (this.isToday(day)) className += " today-cell";
      if (this.isFuture(day)) className += " future-cell";
      // 早中晚背景色
      const shift = this.scheduleMap[day];
      if (shift === "早") className += " morning";
      if (shift === "中") className += " afternoon";
      if (shift === "晚") className += " evening";
      return className;
      return className;
    },

    // ======================== 排班/签到逻辑 ========================
    // 获取日期的班次
    getShift(day) {
      return this.scheduleMap[day];
    },
    // 获取班次样式
    getShiftClass(day) {
      const shift = this.getShift(day);
      if (shift === "早") return "morning";
      if (shift === "中") return "afternoon";
      if (shift === "晚") return "evening";
      return "";
    },
    // 判断是否已签到
    isSigned(day) {
      return this.signDates.includes(day);
    },

    // ======================== 解析work_date字段 ========================
    // 解析后端返回的work_date字符串，拆分排班和签到
    parseWorkDate(workDateStr) {
      this.scheduleMap = {};
      this.signDates = [];
      const str = String(workDateStr || '');
      if (!str.trim()) return;

      const itemList = str.split(",");
      itemList.forEach(item => {
        if (!item.trim()) return;
        // 排班：@2026-04-08_早
        if (item.startsWith("@")) {
          const [date, shift] = item.substring(1).split("_");
          if (date && shift) this.scheduleMap[date] = shift;
        } else {
          // 签到：2026-04-08
          this.signDates.push(item.trim());
        }
      });
      // 自动统计本月数据
      this.calcMonthData();
    },

    // 统计本月考勤数据
    calcMonthData() {
      const year = this.currentMonth.getFullYear();
      const month = this.currentMonth.getMonth() + 1;
      // 本月所有排班日期
      const monthScheduleList = Object.keys(this.scheduleMap).filter(day => {
        const dayYear = new Date(day).getFullYear();
        const dayMonth = new Date(day).getMonth() + 1;
        return dayYear === year && dayMonth === month;
      });
      // 本月已签到日期
      const monthSignList = this.signDates.filter(day => {
        const dayYear = new Date(day).getFullYear();
        const dayMonth = new Date(day).getMonth() + 1;
        return dayYear === year && dayMonth === month;
      });

      this.scheduleCount = monthScheduleList.length;
      this.signCount = monthSignList.length;
      // 缺勤=本月已过的排班日期 - 已签到
      const pastScheduleList = monthScheduleList.filter(day => !this.isFuture(day));
      this.absentCount = pastScheduleList.length - monthSignList.length;
    },

    // ======================== 核心操作：签到 ========================
    handleSign(data) {
      const day = data.day;
      // 二次校验
      if (this.isFuture(day)) {
        this.$notify.error({ title: "签到失败", message: "不能签到未来日期！" });
        return;
      }
      if (!this.getShift(day)) {
        this.$notify.error({ title: "签到失败", message: "今日未排班，无法签到！" });
        return;
      }
      if (this.isSigned(day)) {
        this.$notify.info({ title: "提示", message: "今日已完成签到，无需重复操作！" });
        return;
      }

      // 确认签到
      this.$confirm(`确定签到【${day} ${this.getShift(day)}班】？`, "签到确认", {
        type: "warning"
      }).then(() => {
        // 调用签到接口
        this.postRequest("/system/hr/wordDate", { date: day }).then(resp => {
          if (resp) {
            this.$notify.success({ title: "签到成功", message: `${day} 签到完成！` });
            // 刷新数据
            this.initEmpAttendance();
          }
        });
      });
    },

    // ======================== 初始化加载员工考勤数据 ========================
    initEmpAttendance() {
      let url = "/personnel/ec/init?page="+ this.page + '&size=' + this.size+"&id="+JSON.parse(window.sessionStorage.getItem("user")).employeeId;
      /*if (this.keyword) url += `&name=`+this.keyword;*/

      this.getRequest(url).then(resp => {
        if (resp && resp.data) {
          this.emps = resp.data;
          // 解析第一个员工的考勤数据（搜索后对应员工）
          const workDateStr = this.emps[0].hr.workDates || "";
          this.parseWorkDate(workDateStr);
        }
      });
    },
  },
};
</script>