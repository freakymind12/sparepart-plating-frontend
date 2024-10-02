<template>
  <div class="wrapper-widget">
    <a-flex justify="space-between" style="margin-bottom: 0.3rem">
      <h3 style="font-size: 1.2rem">{{  activeView }} {{ props.title }} Sparepart</h3>
      <a-flex gap="small">
      <a-date-picker v-model:value="yearmonth" v-if="activeView == 'History'" placeholder="Select Month" picker="month" style="height: 25px; width: 100px;"/>
      <a-dropdown placement="bottomRight">
        <a-button size="small" shape="rounded">
          <MenuOutlined style="font-size: small;"/>
        </a-button>
        <template #overlay>
          <a-menu>
            <a-menu-item @click="setView('History')">
              <FolderOpenOutlined/> History
            </a-menu-item>
            <a-menu-item @click="setView('Report')">
              <AreaChartOutlined /> Report
            </a-menu-item>
          </a-menu>
        </template>
      </a-dropdown>
      </a-flex>
    </a-flex>
    <div v-if="activeView === 'History'">
      <CardHistory :category="props.title" :yearmonth="yearmonthFormatted" :refresh="props.refresh" @refresh-completed="emits('refresh-completed')"/>
    </div>
    <div v-else-if="activeView === 'Report'">
      <CardReport/>
    </div>
  </div>
</template>

<script setup>
import { AreaChartOutlined, FolderOpenOutlined, MenuOutlined } from '@ant-design/icons-vue'
import { ref, watch } from 'vue';
import CardHistory from './CardHistory.vue';
import CardReport from './CardReport.vue';
import dayjs from 'dayjs';

const yearmonth = ref(dayjs())
const yearmonthFormatted = ref(dayjs().format('YYYY-MM'))
const activeView = ref('History')
const props = defineProps({
  title:String,
  refresh:Boolean
})
const emits = defineEmits(['refresh-completed'])
const setView = (view) => {
  activeView.value = view;
}

watch(yearmonth, (newVal) => {
  yearmonthFormatted.value = dayjs(newVal).format('YYYY-MM')
})
</script>

<style scoped>
.wrapper-widget {
  width: 600px;
  height: 346px;
  border: 1px solid #333;
  border-radius: 10px;
  padding: 1rem;
}
</style>
