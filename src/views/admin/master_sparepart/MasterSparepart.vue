<template>
  <a-card size="small" class="card-master">
    <a-flex justify="space-between" align="center" style="margin-bottom: 5px">
      <h3>Master Sparepart</h3>
      <a-flex justify="flex-end" gap="small">
        <a-input
          v-model:value="search"
          placeholder="Search.."
          style="width: 70%; min-width: 5%"
          size="small"
        >
          <template #prefix>
            <SearchOutlined />
          </template>
        </a-input>
        <a-button shape="circle" size="small" @click="handleAdd()"><FileAddOutlined /></a-button>
      </a-flex>
    </a-flex>
    <div class="scrollable-table">
      <a-table
        :dataSource="filteredSparepartList"
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
                @confirm="handleDelete(record.sparepart_id)"
              >
                <a-button shape="circle" size="small" danger><DeleteOutlined /></a-button>
              </a-popconfirm>
            </a-space>
          </template>
          <template v-else-if="column.key === 'unit_price'">
            <span>
              {{ record.unit_price.toLocaleString() }}
            </span>
          </template>
          <template v-else-if="column.key === 'reorder_point'">
            <span>
              {{ record.reorder_point.toLocaleString() }}
            </span>
          </template>
          <template v-else-if="column.key === 'safety_stock'">
            <span>
              {{ record.safety_stock.toLocaleString() }}
            </span>
          </template>
        </template>
      </a-table>
    </div>
  </a-card>
  <ModalSparepart
    :visible="modalData.modalVisible"
    :data="modalData.categoryData"
    :mode="modalData.mode"
    @close="handleClose"
    @edit="fetchSparepart"
    @add="fetchSparepart"
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
import ModalSparepart from './ModalSparepart.vue'

const search = ref('')
const sparepartList = ref([])
const columns = [
  {
    title: 'Part ID',
    dataIndex: 'sparepart_id',
    key: 'sparepart_id',
    sorter: (a, b) => a.sparepart_id.localeCompare(b.sparepart_id)
  },
  {
    title: 'Name',
    dataIndex: 'name',
    key: 'name',
    sorter: (a, b) => a.name.localeCompare(b.name)
  },
  {
    title: 'Price',
    dataIndex: 'unit_price',
    key: 'unit_price',
    sorter: (a, b) => a.unit_price - b.unit_price
  },
  {
    title: 'Safety Stock',
    dataIndex: 'safety_stock',
    key: 'safety_stock',
    sorter: (a, b) => a.safety_stock - b.safety_stock
  },
  {
    title: 'Reorder Point',
    dataIndex: 'reorder_point',
    key: 'reorder_point',
    sorter: (a, b) => a.reorder_point - b.reorder_point
  },
  {
    title: 'Category',
    dataIndex: 'category_id',
    key: 'category_id',
    sorter: (a, b) => a.category_id.localeCompare(b.category_id)
  },
  {
    title: 'Description',
    dataIndex: 'description',
    key: 'description',
    sorter: (a, b) => a.description.localeCompare(b.description),
    width: 250
  },
  {
    title: 'Supplier',
    dataIndex: 'supplier_name',
    key: 'supplier_name',
    sorter: (a, b) => a.supplier_name.localeCompare(b.supplier_name)
  },
  {
    title: 'Added By',
    dataIndex: 'username',
    key: 'username',
    sorter: (a, b) => a.username.localeCompare(b.username)
  },
  {
    title: 'Action',
    key: 'action', align:'center'
  }
]
const modalData = ref({
  modalVisible: false,
  mode: null,
  sparepartData: null
})

const filteredSparepartList = computed(() => {
  if (!search.value) {
    return sparepartList.value
  }
  return sparepartList.value.filter(
    (sparepart) =>
      sparepart.sparepart_id.toLowerCase().includes(search.value.toLowerCase()) ||
      sparepart.name.toLowerCase().includes(search.value.toLowerCase()) ||
      sparepart.supplier_name.toLowerCase().includes(search.value.toLowerCase()) ||
      sparepart.username.toLowerCase().includes(search.value.toLowerCase()) ||
      sparepart.description.toLowerCase().includes(search.value.toLowerCase()) ||
      sparepart.category_id.toLowerCase().includes(search.value.toLowerCase())
  )
})

const fetchSparepart = async () => {
  try {
    const response = await axios.get('http://192.168.148.125:5003/api/sparepart')
    sparepartList.value = response.data.data
  } catch (error) {
    console.error(error)
  }
}

const handleDelete = async (id) => {
  try {
    await axios.delete(`http://192.168.148.125:5003/api/sparepart/${id}`)
    fetchSparepart()
    message.success('Success delete sparepart')
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
  await fetchSparepart()
})
</script>

<style scoped>
.card-master {
  width: 100%;
  height: 350px;
  border: 1px solid #555;
}

.scrollable-table {
  height: 280px;
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
