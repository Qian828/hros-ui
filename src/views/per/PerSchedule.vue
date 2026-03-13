<template>
  <div>
    <template>
      <el-tabs v-model="activeName"
               @tab-click="handleClick">
        <el-tab-pane label="排班管理"
                     name="WorkSchedule">
          <WorkSchedule></WorkSchedule>
        </el-tab-pane>


      </el-tabs>
    </template>
  </div>
</template>

<script>
import WorkSchedule from "../../components/per/ec/WorkSchedule.vue";
export default {
  name: "PerSchedule",
  data () {
    return {
      activeName: 'WorkSchedule',
      data: {},
    }
  },
  methods: {

  },
  components: {
    WorkSchedule,
  }
}
</script>

<style scoped>
</style>
