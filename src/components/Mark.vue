<template>
    <el-dialog align-center style="width: 95vw;max-width: 600px;max-height: 85vh;" v-model="show" title="排行榜 (功能已禁用)">
      <div style="text-align: center; padding: 20px;">
        <el-alert
          title="功能不可用"
          type="warning"
          description="由于后端服务未开源，排行榜功能暂时无法使用。"
          show-icon
          :closable="false"
        />
        <el-button style="margin-top: 20px;" @click="show = false">关闭</el-button>
      </div>
    </el-dialog>
</template>

<script lang="ts" setup>
import { ref, watch } from 'vue'

const props = defineProps({
  show: {type: Boolean, required: true},
  loginInfo: {type: Object, required: true}
})

const show = ref(props.show)

watch(() => props.show, (newVal) => {
  show.value = newVal
})

watch(show, (newVal) => {
  if (props.show) (props.show as any).show = newVal;
})

// 移除所有与后端API交互的逻辑
// const mark:Ref<any>=ref([])
// const api =async(args:string[][])=>{ ... }
// const refreshMark=async()=>{ ... }
// const typeMatch=(str:string)=>{ ... }
// function formatter(...) { ... }
// const timeformatter=(n:number)=>{ ... }
</script>

<style scoped>
/* 移除所有与表格和标签相关的样式 */
</style>