<template>
  <a-card size="small" class="card-master">
    <a-flex justify="space-between" align="center" style="margin-bottom: 5px">
      <h3>Master Supplier</h3>
      <a-flex justify="flex-end" gap="small">
        <a-input v-model:value="search" placeholder="Search.." style="width: 50%" size="small">
          <template #prefix>
            <SearchOutlined />
          </template>
        </a-input>
        <a-button shape="circle" size="small" @click="handleAdd()"><FileAddOutlined /></a-button>
      </a-flex>
    </a-flex>
    <div class="scrollable-table">
      <a-table
        :dataSource="filteredSupplierList"
        :columns="columns"
        size="small"
        :pagination="false"
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
                @confirm="handleDelete(record.supplier_id)"
              >
                <a-button shape="circle" size="small" danger><DeleteOutlined /></a-button>
              </a-popconfirm>
            </a-space>
          </template>
        </template>
      </a-table>
    </div>
  </a-card>
  <ModalSupplier
    :visible="modalData.modalVisible"
    :data="modalData.categoryData"
    :mode="modalData.mode"
    @close="handleClose"
    @edit="fetchSupplier"
    @add="fetchSupplier"
  />
</template>

<script setup>
import {
  DeleteOutlined,
  EditOutlined,
  FileAddOutlined,
  SearchOutlined
} from '@ant-design/icons-vue'
import axios from 'axios'
import { onMounted, ref, computed } from 'vue'
import { message } from 'ant-design-vue'
import ModalSupplier from './ModalSupplier.vue'

const search = ref('')
const supplierList = ref([])
const columns = [
  {
    title: 'Name',
    dataIndex: 'supplier_name',
    key: 'supplier_name',
    sorter: (a, b) => a.supplier_name.localeCompare(b.supplier_name)
  },
  {
    title: 'Location',
    dataIndex: 'location',
    key: 'location',
    sorter: (a, b) => a.location.localeCompare(b.location)
  },
  {
    title: 'Action',
    key: 'action', align:'center'
  }
]
const modalData = ref({
  modalVisible: false,
  mode: null,
  categoryData: null
})

const fetchSupplier = async () => {
  try {
    const response = await axios.get('http://192.168.148.125:5003/api/supplier')
    supplierList.value = response.data.data
  } catch (error) {
    console.error(error)
  }
}

const filteredSupplierList = computed(() => {
  if (!search.value) {
    return supplierList.value
  }
  return supplierList.value.filter(
    (supplier) =>
      supplier.supplier_name.toLowerCase().includes(search.value.toLowerCase()) ||
      supplier.location.toLowerCase().includes(search.value.toLowerCase())
  )
})

const handleDelete = async (id) => {
  try {
    await axios.delete(`http://192.168.148.125:5003/api/supplier/${id}`)
    fetchSupplier()
    message.success('Success delete supplier')
  } catch (error) {
    console.error(error)
  }
}

const handleEdit = async (data) => {
  modalData.value.modalVisible = true
  modalData.value.categoryData = data
  modalData.value.mode = 'edit'
}

const handleAdd = async () => {
  modalData.value.modalVisible = true
  modalData.value.categoryData = null
  modalData.value.mode = 'add'
}

const handleClose = () => {
  modalData.value.modalVisible = false
}

onMounted(async () => {
  await fetchSupplier()
})
</script>

<style scoped>
.card-master {
  min-width: 33%;
  height: 300px;
  border: 1px solid #555;
}

.scrollable-table {
  height: 220px;
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
