<template>
  <a-modal :footer="null" v-bind:open="props.visible" @cancel="emits('close')" centered>
    <template #title>
      <span>
        <a-tag color="magenta"><ToolOutlined /> {{ props.data.name }} </a-tag>
        <a-tag color="blue"><SettingOutlined /> {{ props.data.sparepart_id }}</a-tag>
        <a-tag color="volcano">Machine : {{ props.data.machine_id }}</a-tag>
      </span>
    </template>
    <div class="wrapper-history-list">
      <a-timeline>
        <a-timeline-item v-for="item in historyList" :key="item.goods_out_id">
          <a-flex wrap="wrap">
            <a-tag><ScheduleOutlined /> {{ item.date_out }}</a-tag>
            <a-tag><UserOutlined /> {{ item.username }}</a-tag>
            <a-tag><TagOutlined /> {{ item.quantity_out }} Unit</a-tag>
          </a-flex>
          <a-tag  style="margin-top: 5px;width:95%;"><span style="font-weight: bold;">Note</span> : {{ item.note }} Unit</a-tag>
        </a-timeline-item>
      </a-timeline>
    </div>
  </a-modal>
</template>

<script setup>
import { onMounted, ref, watch } from 'vue'
import {
  ScheduleOutlined,
  SettingOutlined,
  TagOutlined,
  ToolOutlined,
  UserOutlined
} from '@ant-design/icons-vue'
import axios from 'axios'

const props = defineProps({
  visible: Boolean,
  data: Object
})
const historyList = ref([])
const emits = defineEmits(['close'])

const fetchHistory = async (data) => {
  try {
    const response = await axios.get(
      `http://192.168.148.125:5003/api/sparepart-outgoing?year=${data.yearmonth.split('-')[0]}&month=${data.yearmonth.split('-')[1]}&sparepart_id=${data.sparepart_id}&machine_id=${data.machine_id}`
    )
    historyList.value = response.data.data
  } catch (error) {
    console.error(error)
  }
}

onMounted(() => {
  fetchHistory(props.data)
})

watch(props, (newVal) => {
  fetchHistory(newVal.data)
})
</script>

<style scoped>
.wrapper-history-list {
  padding-top: 20px;
  max-height: 200px;
  height: 200px;
  overflow: auto;

}

.wrapper-history-list::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.wrapper-history-list::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.wrapper-history-list::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 10px;
}

.wrapper-history-list::-webkit-scrollbar-thumb:hover {
  background: #555;
}

.ant-modal-title .ant-tag {
  font-size: .85rem;
}

.ant-timeline-item-content .ant-tag {
  font-size: .8rem;
  font-weight: 400;
}
</style>
