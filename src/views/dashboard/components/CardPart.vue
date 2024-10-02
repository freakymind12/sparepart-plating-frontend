<template>
  <a-card class="card-stock" size="small" :bordered="false">
    <a-flex justify="flex-end" style="margin-bottom: 0.5rem">
      <a-tag>Last Updated : {{ lastUpdate }}</a-tag>
      <a-input
        v-model:value="search"
        placeholder="Search.."
        style="width: 150px; height: 20px"
        size="small"
      >
        <template #prefix>
          <SearchOutlined />
        </template>
      </a-input>
    </a-flex>
    <div class="wrapper-list" v-if="filteredList.length > 0">
      <a-timeline style="margin-top: 0.5rem">
        <a-timeline-item
          :color="item.actual_stock <= item.reorder_point ? 'red' : 'green'"
          v-for="item in filteredList"
          :key="item.sparepart_id"
        >
          <template #dot>
            <template v-if="item.actual_stock <= item.reorder_point">
              <CloseCircleOutlined style="font-size: large;"/>
            </template>
            <template v-else>
              <CheckCircleOutlined style="font-size: large;"/>
            </template>
          </template>
          <a-flex vertical gap="small">
            <a-tag color="blue"><ToolOutlined /> {{ item.name }}</a-tag>
            <a-tag color="magenta"><SettingOutlined /> {{ item.sparepart_id }}</a-tag>
            <a-flex>
              <a-tag color="orange">Stock : {{ item.actual_stock }}</a-tag>
              <a-tag color="cyan">Reorder : {{ item.reorder_point }}</a-tag>
              <a-tag color="purple">Safety : {{ item.safety_stock }}</a-tag>
              <a-tag class="hover-effect" @click="handleAdd(item)"><FormOutlined /></a-tag>
            </a-flex>
          </a-flex>
        </a-timeline-item>
      </a-timeline>
    </div>
    <a-empty v-else style="min-height: 570px; padding-top: 50%" />
  </a-card>
  <ModalOutgoing
    :visible="modalData.modalVisible"
    :data="modalData.outgoingData"
    :mode="modalData.mode"
    @close="handleClose"
    @add="fetchAndEmit()"
  />
</template>

<script setup>
import {
  CheckCircleOutlined,
  CloseCircleOutlined,
  FormOutlined,
  SearchOutlined,
  SettingOutlined,
  ToolOutlined
} from '@ant-design/icons-vue'
import axios from 'axios'
import { onMounted, ref, computed } from 'vue'
import ModalOutgoing from './ModalOutgoing.vue'
import { message } from 'ant-design-vue'

const search = ref('')
const lastUpdate = ref('')
const itemList = ref([])
const modalData = ref({
  modalVisible: false,
  mode: null,
  outgoingData: null
})
const props = defineProps({
  category: String
})
const emits = defineEmits(['add'])

const fetchSparepart = async () => {
  try {
    const response = await axios.get(
      `http://192.168.148.125:5003/api/sparepart?category_id=${props.category}`
    )
    itemList.value = response.data.data
    lastUpdate.value = itemList.value[0].query_date
  } catch (error) {
    console.error(error)
  }
}

const fetchAndEmit = () => {
  fetchSparepart()
  emits('add')
}

const filteredList = computed(() => {
  if (!search.value) {
    return itemList.value
  }
  return itemList.value.filter(
    (item) =>
      item.sparepart_id.toLowerCase().includes(search.value.toLowerCase()) ||
      item.name.toLowerCase().includes(search.value.toLowerCase())
  )
})

onMounted(() => {
  fetchSparepart()
})

const handleAdd = async (item) => {
  item.actual_stock === 0 ? message.error(`There is no stock for this sparepart`) : 
  modalData.value.modalVisible = true
  modalData.value.outgoingData = item
  modalData.value.mode = 'add'
}

const handleClose = () => {
  modalData.value.modalVisible = false
}
</script>

<style scoped>
.card-stock {
  min-height: 570px;
}

.wrapper-list {
  padding-left: 0.3rem;
  padding-top: 0.2rem;
  height: 570px;
  overflow: auto;
  overflow-x: hidden;
  padding-right: 0.5rem;
}

.wrapper-list::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.wrapper-list::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.wrapper-list::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 10px;
}

.wrapper-list::-webkit-scrollbar-thumb:hover {
  background: #555;
}

.ant-tag {
  text-wrap: wrap;
  font-weight: 500;
  font-size: 13px;
}

.hover-effect {
  display: inline-block;
  transition:
    transform 0.2s,
    box-shadow 0.2s;
  border-radius: 4px;
  cursor: pointer;
}

.hover-effect:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.2);
}

</style>
