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


        <div class="stat-box">
          <span class="stat-item sign">{{ username }}本月排班：{{ scheduleCount }}天</span>
        </div>
      </div>
    </div>

    <!-- 日历主体 -->
    <el-calendar v-model="currentMonth" :first-day-of-week="7">
      <template slot="dateCell" slot-scope="{ data }">
        <div class="calendar-cell" :class="getCellClass(data.day)">

          <!-- 左侧内容：日期 + 班次 + 状态 -->
          <div class="cell-left">
            <div class="day-text">
              {{ data.day.split('-').slice(2).join('-') }}
              <div v-if="getShift(data.day)" class="shift-tag" :class="getShiftClass(data.day)">
                {{ getShift(data.day) }}班
              </div>
            </div>



            <div v-if="getShift(data.day)" class="signed-tag">
              已排班
            </div>

            <div v-else-if="hasEmployee && isPast(data.day)" class="no-schedule">
              未排班
            </div>
          </div>

          <!-- 右侧按钮：置顶靠右 -->
          <div class="cell-right">
            <el-button
                v-if="getShift(data.day) && !isPast(data.day)"
                type="danger"
                size="mini"
                icon="el-icon-remove"
                class="cancel-btn"
                @click="cancelSchedule(data.day, getShift(data.day))"
            >
              取消排班
            </el-button>

            <el-button
                v-if="hasEmployee && !isPast(data.day) && !getShift(data.day)"
                type="primary"
                size="medium"
                icon="el-icon-plus"
                class="sign-btn-improved"
                @click="open(data)"
            >
              排班
            </el-button>
          </div>

        </div>
      </template>
    </el-calendar>
  </div>
</template>

<style scoped>
/* 日历单元格 - 左右布局 */
.calendar-cell {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  min-height: 60px;
  padding: 4px;
  position: relative;
}

/* 左侧：日期 + 班次 + 状态 */
.cell-left {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

/* 右侧：按钮置顶 */
.cell-right {
  display: flex;
  align-items: flex-start;
  padding-top: 2px;
}

/* 取消排班按钮 */
.cancel-btn {
  font-size: 12px !important;
  padding: 8px 8px !important;
  height: 50px !important;
  font-weight: 500 !important;
}

/* 排班按钮 */
.sign-btn-improved {
  font-size: 12px !important;
  padding: 4px 8px !important;
  height: 50px !important;
  font-weight: 500 !important;
}

/* 顶部统计栏 */
.stat-box {
  display: flex;
  gap: 20px;
}
.absent-tag {
  color: #722ed1;
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 2px;
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
  font-weight: bold;
}
.stat-item.absent {
  background: #fff2f0;
  color: #ff4d4f;
  font-weight: bold;
}

.day-text {
  font-weight: bold;
  font-size: 14px;
}
.today-cell {
  background: #e6f7ff !important;
  border-radius: 4px;
}
.future-cell {
  opacity: 0.5;
}

/* 班次标签 */
.shift-tag {
  font-size: 12px;
  padding: 2px 6px;
  border-radius: 3px;
  display: inline-block;
  width: fit-content;
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

.signed-tag {
  color: #f5222d;
  font-size: 12px;
  font-weight: bold;
}
.no-schedule {
  color: #999;
  font-size: 12px;
}

.calendar-cell.morning {
  background: #f6ffed !important;
}
.calendar-cell.afternoon {
  background: #fff7e6 !important;
}
.calendar-cell.evening {
  background: #e0f7fa !important;
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
  font-size: 16px;
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
      username:"",
      userId:0,
      currentMonth: new Date(), // 当前日历月份
      emps: [],
      total: 0,
      page: 1,
      hasEmployee: false,
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
    this.init();
  },
  methods: {
    cancelSchedule(day, shift) {
      const fullDate = "@" + day + "_" + shift;
      this.$confirm("确定要取消【"+this.username+"】此日期的排班吗？", "提示", {
        type: "warning"
      }).then(() => {
        this.getRequest("/system/hr/cancelSchedule?id=" + this.userId + "&wordDate=" + encodeURIComponent(fullDate))
            .then(res => {
              this.$message.success("取消成功");
              this.init();
            });
      }).catch(() => {
        this.$message.info("已取消操作");
      });
    },
    open(data) {
      this.$msgbox({
        title: "请选择" + this.username + "【" + data.day + "】的班次",
        dangerouslyUseHTMLString: true,
        message: `
      <div style="padding: 20px; text-align: center;">
        <label style="margin:0 20px; font-size:14px;">
          <input type="radio" name="shift" value="早" /> 早班
        </label>
        <label style="margin:0 20px; font-size:14px;">
          <input type="radio" name="shift" value="中" /> 中班
        </label>
        <label style="margin:0 20px; font-size:14px;">
          <input type="radio" name="shift" value="晚" /> 晚班
        </label>
      </div>
    `,
        showCancelButton: true,
        confirmButtonText: "确定排班",
        cancelButtonText: "取消",
        closeOnClickModal: false,
      }).then(() => {
        // 点确定才进这里
        let checked = document.querySelector('input[name="shift"]:checked');

        if (!checked) {
          this.$message.warning("请选择一个班次！");
        }

        let shift = checked.value;


        this.getRequest("/system/hr/wordDateSchedule?id="+this.userId+"&wordDate="+"@" + data.day + "_" + shift).then(res => {
          this.$message.success(`排班成功：${shift}班`);
          this.init();
        });

      }).catch(() => {
        // 点取消才进这里
        this.$message.info("已取消排班");
      });
    },

    // 判断是否是过去日期（今天之前）
    isPast(day) {
      const todayStr = new Date().toISOString().split('T')[0];
      return day < todayStr;
    },
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

    clear(){
      this.scheduleMap = {}        // 清空排班
      this.signDates = []          // 清空签到
      this.scheduleCount = 0       // 清空统计
      this.signCount = 0
      this.absentCount = 0
      this.emps = []
    },
    init(){
      this.scheduleMap = {}        // 清空排班
      this.signDates = []          // 清空签到
      this.scheduleCount = 0       // 清空统计
      this.signCount = 0
      this.absentCount = 0
      this.emps = []
      this.username = ""

      let user = JSON.parse(window.sessionStorage.getItem("user"));
      this.username = user.username;
      let url = "/personnel/ec/init?page="+ this.page + '&size=' + this.size+"&id="+(user.employeeId || '');

      this.getRequest(url).then(resp => {
        if (resp && resp.data) {
          this.emps = resp.data;
          this.hasEmployee = true;
          const currentEmployeeId = user.employeeId; // 当前登录人ID
          const pageEmployeeId = this.emps[0].id;    // 页面显示的员工ID

          // 只有【自己】才能看到签到按钮
          this.isCurrentUser = currentEmployeeId === pageEmployeeId;
          this.username = this.emps[0].name;
          this.userId = this.emps[0].hr.id;
          // 解析第一个员工的考勤数据（搜索后对应员工）
          const workDateStr = this.emps[0].hr.workDates || "";
          this.parseWorkDate(workDateStr);
        }
      });
    },
    // ======================== 初始化加载员工考勤数据 ========================
    initEmpAttendance() {
      this.scheduleMap = {}        // 清空排班
      this.signDates = []          // 清空签到
      this.scheduleCount = 0       // 清空统计
      this.signCount = 0
      this.absentCount = 0
      this.emps = []
      this.username = ""

      let url = "/personnel/ec/init?page="+ this.page + '&size=' + this.size;
      if (this.keyword) url += `&name=`+this.keyword;

      this.getRequest(url).then(resp => {
        if (resp && resp.data) {
          this.emps = resp.data;
          if (!resp || !resp.data || resp.data.length === 0) {
            this.emps = []
            this.username = ""
            this.scheduleMap = {}
            this.signDates = []
            this.scheduleCount = 0
            this.signCount = 0
            this.absentCount = 0
            this.hasEmployee = false
            this.$message.warning("未查询到该员工信息");
            return;
          }
          this.hasEmployee = true;
          this.username = this.emps[0].name;
          this.userId = this.emps[0].hr.id;
          // 解析第一个员工的考勤数据（搜索后对应员工）
          const workDateStr = this.emps[0].hr.workDates || "";
          this.parseWorkDate(workDateStr);
        }
      });
    },
  },
};
</script>