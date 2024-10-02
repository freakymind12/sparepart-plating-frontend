<template>
  <a-card size="small" class="card-master">
    <a-flex justify="space-between" align="center" style="margin-bottom: 5px">
      <h3>Master Category</h3>
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
        :dataSource="filteredCategoryList"
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
                @confirm="handleDelete(record.category_id)"
              >
                <a-button shape="circle" size="small" danger><DeleteOutlined /></a-button>
              </a-popconfirm>
            </a-space>
          </template>
        </template>
      </a-table>
    </div>
  </a-card>
  <ModalCategory
    :visible="modalData.modalVisible"
    :data="modalData.categoryData"
    :mode="modalData.mode"
    @close="handleClose"
    @edit="fetchCategory"
    @add="fetchCategory"
  />
</template>

<script setup>
import {
  DeleteOutlined,
  EditOutlined,
  FileAddOutlined,
  SearchOutlined
} from '@ant-design/icons-vue'
import { message } from 'ant-design-vue'
import axios from 'axios'
import { onMounted, ref, computed } from 'vue'
import ModalCategory from './ModalCategory.vue'

const search = ref('')
const categoryList = ref([])
const columns = [
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
    sorter: (a, b) => a.description.localeCompare(b.description)
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

const fetchCategory = async () => {
  try {
    const response = await axios.get('http://192.168.148.125:5003/api/category')
    categoryList.value = response.data.data
  } catch (error) {
    console.error(error)
  }
}

// Computed property untuk data yang sudah difilter berdasarkan inputan pencarian
const filteredCategoryList = computed(() => {
  if (!search.value) {
    return categoryList.value
  }
  return categoryList.value.filter(
    (category) =>
      category.category_id.toLowerCase().includes(search.value.toLowerCase()) ||
      category.description.toLowerCase().includes(search.value.toLowerCase())
  )
})

const handleDelete = async (id) => {
  try {
    await axios.delete(`http://192.168.148.125:5003/api/category/${id}`)
    fetchCategory()
    message.success('Success delete category')
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
  await fetchCategory()
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
