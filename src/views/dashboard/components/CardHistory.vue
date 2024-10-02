<template>
  <a-collapse
    v-model:activeKey="activeKey"
    style="background: rgb(255, 255, 255)"
    v-if="reportList.length > 0"
  >
    <template #expandIcon="{ isActive }">
      <caret-right-outlined :rotate="isActive ? 90 : 0" />
    </template>
    <a-collapse-panel v-for="data in reportList" :key="data.sparepart_id">
      <template #header>
        <a-flex justify="space-between">
          <a-flex>
            <a-tag color="blue"> <SettingOutlined /> {{ data.sparepart_id }} </a-tag>
            <a-tag color="magenta"> <ToolOutlined /> {{ data.sparepart_name }} </a-tag>
          </a-flex>
          <a-tag color="cyan" style="width: 60px;"> Total : {{ data.total_used }}</a-tag>
        </a-flex>
      </template>
      <a-flex wrap="wrap">
        <a-tag
          color="volcano"
          v-for="item in data.used_history"
          :key="item.machine_id"
          class="machine-used"
          @click="
            seeModalHistory({ ...item, sparepart_id: data.sparepart_id, name: data.sparepart_name })
          "
          ><PartitionOutlined /> {{ item.machine_id }} : {{ item.used }}</a-tag
        >
      </a-flex>
    </a-collapse-panel>
  </a-collapse>
  <a-flex v-else justify="center" align="center" style="padding-top: 10%">
    <a-empty></a-empty>
  </a-flex>
  <ModalInfoHistory :visible="modalData.visible" v-if="reportList.length>0" :data="modalData.data" @close="handleClose" />
</template>

<script setup>
import {
  CaretRightOutlined,
  PartitionOutlined,
  SettingOutlined,
  ToolOutlined
} from '@ant-design/icons-vue'
import axios from 'axios'
import { onMounted, ref, watch } from 'vue'
import ModalInfoHistory from './ModalInfoHistory.vue'

const activeKey = ref([])
const reportList = ref([])
const props = defineProps({
  category: String,
  yearmonth: String,
  refresh:Boolean
})
const modalData = ref({
  visible: false,
  data: null
})
const emits = defineEmits(['refresh-completed'])


const fetchReportSparepart = async (data) => {
  try {
    const response = await axios.get(
      `http://192.168.148.125:5003/api/sparepart-outgoing/report?category_id=${data.category}&year=${data.yearmonth.split('-')[0]}&month=${data.yearmonth.split('-')[1]}`
    )
    reportList.value = response.data.data
  } catch (error) {
    console.error(error)
  }
}

const seeModalHistory = (item) => {
  modalData.value.visible = true
  modalData.value.data = item
}

const handleClose = () => {
  modalData.value.visible = false
}

onMounted(async () => {
  await fetchReportSparepart(props)
})

watch(props, (newVal) => {
  fetchReportSparepart(newVal)
  if(newVal.refresh === true){
    emits('refresh-completed')
  }
},
{ deep: true })
</script>

<style scoped>

.machine-used {
  transition: transform 0.3s ease;
}

.machine-used:hover {
  cursor: pointer;
  transform: scale(1.1);
}

.ant-tag {
  font-size: 12px;
  font-weight: 500;
}
</style>
