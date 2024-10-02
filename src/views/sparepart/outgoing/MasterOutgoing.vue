<template>
  <a-card class="card-master" size="small">
    <template #title>
      <span style="font-size: large;">Outgoing Sparepart</span>
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
        :dataSource="filteredOutgoingList"
        :columns="columns"
        size="small"
        :pagination="false"
        v-if="filteredOutgoingList.length > 0"
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
                @confirm="handleDelete(record.goods_out_id)"
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
  <ModalOutgoing
    :visible="modalData.modalVisible"
    :data="modalData.outgoingData"
    :mode="modalData.mode"
    @close="handleClose"
    @edit="fetchOutgoing"
    @add="fetchOutgoing"
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
import ModalOutgoing from './ModalOutgoing.vue';

const search = ref('')
const outgoingList = ref([])
const columns = [
  {
    title: 'Date In',
    dataIndex: 'date_out',
    key: 'date_out',
    sorter: (a, b) => a.date_out.localeCompare(b.date_out)
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
    dataIndex: 'quantity_out',
    key: 'quantity_out',
    sorter: (a, b) => a.quantity_out.localeCompare(b.quantity_out)
  },
  {
    title: 'Machine',
    dataIndex: 'machine_id',
    key: 'machine_id',
    sorter: (a, b) => a.machine_id.localeCompare(b.machine_id)
  },
  {
    title: 'Note',
    dataIndex: 'note',
    key: 'note',
    sorter: (a, b) => a.note.localeCompare(b.note),
    width:150
  },
  {
    title: 'PIC',
    dataIndex: 'username',
    key: 'username',
    sorter: (a, b) => a.username.localeCompare(b.username),
    width:50
  },
  {
    title: 'Action',
    key: 'action', align:'center'
  }
]
const modalData = ref({
  modalVisible: false,
  mode: null,
  outgoingData: null
})

const handleDelete = async (id) => {
  try {
    await axios.delete(`http://192.168.148.125:5003/api/sparepart-outgoing/${id}`)
    fetchOutgoing()
    message.success('Success delete outgoing')
  } catch (error) {
    console.error(error)
  }
}

const handleEdit = async (data) => {
  modalData.value.modalVisible = true
  modalData.value.outgoingData = data
  modalData.value.mode = 'edit'
}

const handleAdd = async () => {
  modalData.value.modalVisible = true
  modalData.value.outgoingData = null
  modalData.value.mode = 'add'
}

const handleClose = () => {
  modalData.value.modalVisible = false
}

const filteredOutgoingList = computed(() => {
  if (!search.value) {
    return outgoingList.value
  }
  return outgoingList.value.filter(
    (outgoing) =>
      outgoing.name.toLowerCase().includes(search.value.toLowerCase()) ||
      outgoing.date_out.toLowerCase().includes(search.value.toLowerCase()) ||
      outgoing.machine_id.toLowerCase().includes(search.value.toLowerCase()) ||
      outgoing.sparepart_id.toLowerCase().includes(search.value.toLowerCase()) || 
      outgoing.username.toLowerCase().includes(search.value.toLowerCase()) || 
      outgoing.quantity_out.toString().includes(search.value)
  )
})

const fetchOutgoing = async () => {
  try {
    const response = await axios.get('http://192.168.148.125:5003/api/sparepart-outgoing')
    outgoingList.value = response.data.data
  } catch (error) {
    console.error(error)
  }
}

onMounted(async () => {
  await fetchOutgoing()
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
