<template>
  <a-card class="card-master" size="small">
    <template #title>
      <span style="font-size: large;">Incoming Sparepart</span>
    </template>
    <template #extra>
      <a-flex gap="small">
        <a-input v-model:value="search" placeholder="Search.." style="width: 120px" size="small">
          <template #prefix>
            <SearchOutlined />
          </template>
        </a-input>
        <a-button shape="circle" size="small" @click="handleAdd()"><FileAddOutlined /></a-button>
      </a-flex>
    </template>
    <div class="scrollable-table">
      <a-table
        :dataSource="filteredIncomingList"
        :columns="columns"
        size="small"
        :pagination="false"
        v-if="filteredIncomingList.length > 0"
      >
        <template #bodyCell="{ column, record }">
          <template v-if="column.key === 'action'">
            <a-space>
              <a-button shape="circle" size="small" @click="handleEdit(record)"
                ><EditOutlined
              /></a-button>
              <a-popconfirm
                title="Are you sure delete this data?"
                ok-text="Yes"
                cancel-text="No"
                @confirm="handleDelete(record.goods_in_id)"
              >
                <a-button shape="circle" size="small" danger><DeleteOutlined /></a-button>
              </a-popconfirm>
            </a-space>
          </template>
        </template>
      </a-table>
        <a-empty v-else style="padding-top: 20%;" description="There is no data with this search key"></a-empty>
    </div>
  </a-card>
  <ModalIncoming
    :visible="modalData.modalVisible"
    :data="modalData.incomingData"
    :mode="modalData.mode"
    @close="handleClose"
    @edit="fetchIncoming"
    @add="fetchIncoming"
  />
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import {
  FileAddOutlined,
  SearchOutlined,
  EditOutlined,
  DeleteOutlined
} from '@ant-design/icons-vue'
import axios from 'axios'
import { message } from 'ant-design-vue';
import ModalIncoming from './ModalIncoming.vue';


const search = ref('')
const incomingList = ref([])
const columns = [
  {
    title: 'Date In',
    dataIndex: 'date_in',
    key: 'date_in',
    sorter: (a, b) => a.date_in.localeCompare(b.date_in)
  },
  {
    title: 'Sparepart ID',
    dataIndex: 'sparepart_id',
    key: 'sparepart_id',
    sorter: (a, b) => a.sparepart_id.localeCompare(b.sparepart_id),
    width:150
  },
  {
    title: 'Name',
    dataIndex: 'name',
    key: 'name',
    sorter: (a, b) => a.name.localeCompare(b.name),
    width:150
  },
  {
    title: 'Qty',
    dataIndex: 'quantity_in',
    key: 'quantity_in',
    sorter: (a, b) => a.quantity_in.localeCompare(b.quantity_in)
  },
  {
    title: 'Supplier',
    dataIndex: 'supplier_name',
    key: 'supplier_name',
    sorter: (a, b) => a.supplier_name.localeCompare(b.supplier_name)
  },
  {
    title: 'PIC',
    dataIndex: 'username',
    key: 'username',
    sorter: (a, b) => a.username.localeCompare(b.username),
    width:100
  },
  {
    title: 'Action',
    key: 'action', align:'center'
  }
]
const modalData = ref({
  modalVisible: false,
  mode: null,
  incomingData: null
})

const filteredIncomingList = computed(() => {
  if (!search.value) {
    return incomingList.value
  }
  return incomingList.value.filter(
    (incoming) =>
      incoming.name.toLowerCase().includes(search.value.toLowerCase()) ||
      incoming.date_in.toLowerCase().includes(search.value.toLowerCase()) ||
      incoming.supplier_name.toLowerCase().includes(search.value.toLowerCase()) ||
      incoming.sparepart_id.toLowerCase().includes(search.value.toLowerCase()) || 
      incoming.quantity_in.toString().includes(search.value) || 
      incoming.username.toLowerCase().includes(search.value.toLowerCase())
  )
})

const fetchIncoming = async () => {
  try {
    const response = await axios.get('http://192.168.148.125:5003/api/sparepart-incoming')
    incomingList.value = response.data.data
  } catch (error) {
    console.error(error)
  }
}

const handleDelete = async (id) => {
  try {
    await axios.delete(`http://192.168.148.125:5003/api/sparepart-incoming/${id}`)
    fetchIncoming()
    message.success('Success delete incoming')
  } catch (error) {
    console.error(error)
  }
}

const handleEdit = async (data) => {
  modalData.value.modalVisible = true
  modalData.value.incomingData = data
  modalData.value.mode = 'edit'
}

const handleAdd = async () => {
  modalData.value.modalVisible = true
  modalData.value.incomingData = null
  modalData.value.mode = 'add'
}

const handleClose = () => {
  modalData.value.modalVisible = false
}

onMounted(async () => {
  await fetchIncoming()
})
</script>

<style scoped>
.card-master {
  min-width: 40%;
  height: 600px;
  border: 1px solid #555;
}

.scrollable-table {
  height: 500px;
  overflow: auto;
  padding-right: 8px;
}

.scrollable-table::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

.scrollable-table::-webkit-scrollbar-track {
  background: #f1f1f1;
}

.scrollable-table::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 10px;
}

.scrollable-table::-webkit-scrollbar-thumb:hover {
  background: #555;
}

</style>
