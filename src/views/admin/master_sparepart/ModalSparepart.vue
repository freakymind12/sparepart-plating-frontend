<template>
  <a-modal v-bind:open="props.visible" :footer="null" @cancel="emits('close')" centered>
    <h2>{{ capitalizeEachWord(props.mode + ' sparepart') }}</h2>
    <a-form
      layout="vertical"
      :model="form"
      @finish="props.mode === 'add' ? handleAdd() : handleEdit(props.data.sparepart_id)"
    >
      <a-form-item label="Sparepart ID" name="sparepart_id" :rules="[{ required: true }]">
        <a-input placeholder="Sparepart ID" v-model:value="form.sparepart_id" />
      </a-form-item>
      <a-form-item label="Name" name="name" :rules="[{ required: true }]">
        <a-input placeholder="Name" v-model:value="form.name" />
      </a-form-item>
      <a-form-item label="Price" name="unit_price" :rules="[{ required: true }]">
        <a-input-number
          :min="0"
          placeholder="Price"
          v-model:value="form.unit_price"
          style="width: 100%"
          addon-before="IDR"
          :formatter="formatNumber"
          :parser="parseNumber"
        />
      </a-form-item>
      <a-form-item label="Safety stock" name="safety_stock" :rules="[{ required: true }]">
        <a-input-number
          :min="0"
          placeholder="Safety stock"
          v-model:value="form.safety_stock"
          style="width: 100%"
          addon-before="Qty"
          :formatter="formatNumber"
          :parser="parseNumber"
        />
      </a-form-item>
      <a-form-item label="Reorder point" name="reorder_point" :rules="[{ required: true }]">
        <a-input-number
          :min="0"
          placeholder="Reorder point"
          v-model:value="form.reorder_point"
          style="width: 100%"
          addon-before="Qty"
          :formatter="formatNumber"
          :parser="parseNumber"
        />
      </a-form-item>
      <a-form-item label="Description" name="description" :rules="[{ required: true }]">
        <a-input placeholder="Description" v-model:value="form.description" />
      </a-form-item>
      <a-form-item label="Category" name="category_id" :rules="[{ required: true }]">
        <a-select v-model:value="form.category_id" placeholder="Category" :options="categoryList">
        </a-select>
      </a-form-item>
      <a-form-item label="Supplier" name="supplier_id" :rules="[{ required: true }]">
        <a-select v-model:value="form.supplier_id" placeholder="Supplier" :options="supplierList">
        </a-select>
      </a-form-item>
      <a-form-item>
        <a-flex justify="flex-end">
          <a-space :size="10">
            <a-button type="primary" html-type="submit" :disabled="isSubmitDisabled">
              Submit
            </a-button>
          </a-space>
        </a-flex>
      </a-form-item>
    </a-form>
  </a-modal>
</template>

<script setup>
import { onMounted, ref, watch, computed } from 'vue'
import axios from 'axios'
import { message } from 'ant-design-vue'
import auth from '@/auth/auth'

const form = ref({
  sparepart_id: '',
  name: '',
  unit_price: 0,
  safety_stock: 0,
  reorder_point: 0,
  category_id: '',
  description: '',
  supplier_id: '',
  id_user: ''
})
const emits = defineEmits(['close', 'edit', 'add'])
const props = defineProps({
  visible: Boolean,
  data: Object,
  mode: String
})
const supplierList = ref([])
const categoryList = ref([])

const handleEdit = async (id) => {
  try {
    await axios.patch(`http://192.168.148.125:5003/api/sparepart/${id}`, {
      sparepart_id: form.value.sparepart_id,
      name: form.value.name,
      description: form.value.description,
      unit_price: form.value.unit_price,
      safety_stock: form.value.safety_stock,
      reorder_point: form.value.reorder_point,
      category_id: form.value.category_id,
      supplier_id: form.value.supplier_id
    })
    emits('edit')
    emits('close')
    message.success('Success edit sparepart')
  } catch (error) {
    console.error(error)
  }
}

const handleAdd = async () => {
  try {
    await getUserData()
    await axios.post(`http://192.168.148.125:5003/api/sparepart/`, {
      sparepart_id: form.value.sparepart_id,
      name: form.value.name,
      description: form.value.description,
      unit_price: form.value.unit_price,
      safety_stock: form.value.safety_stock,
      reorder_point: form.value.reorder_point,
      category_id: form.value.category_id,
      supplier_id: form.value.supplier_id,
      id_user: form.value.id_user
    })
    emits('add')
    emits('close')
    message.success('Success add sparepart')
    handleReset()
  } catch (error) {
    console.error(error)
  }
}

const handleReset = () => {
  form.value = {
    sparepart_id: '',
    name: '',
    unit_price: 0,
    safety_stock: 0,
    reorder_point: 0,
    category_id: '',
    description: '',
    supplier_id: ''
  }
}

function capitalizeEachWord(str) {
  return str
    .split(' ')
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
    .join(' ')
}

const formatNumber = (value) => {
  if (!value) return ''
  return value.replace(/\B(?=(\d{3})+(?!\d))/g, ',') // Menambahkan koma setiap 3 digit
}

const parseNumber = (value) => {
  return value.replace(/\$\s?|(,*)/g, '') // Menghapus karakter yang tidak diperlukan
}

const fetchSuppliers = async () => {
  try {
    const response = await axios.get('http://192.168.148.125:5003/api/supplier')
    const data = response.data.data
    supplierList.value = data.map((supplier) => ({
      value: supplier.supplier_id,
      label: supplier.supplier_name
    }))
  } catch (error) {
    console.error(error)
  }
}

const fetchCategory = async () => {
  try {
    const response = await axios.get('http://192.168.148.125:5003/api/category')
    const data = response.data.data
    categoryList.value = data.map((category) => ({
      value: category.category_id,
      label: category.description
    }))
  } catch (error) {
    console.error(error)
  }
}

const getUserData = async () => {
  try {
    const data = await auth.checkRoles()
    form.value.id_user = data.id
  } catch (error) {
    console.error(error)
  }
}

watch(
  () => props.data,
  (newData) => {
    if (newData) {
      form.value.sparepart_id = newData.sparepart_id || ''
      form.value.name = newData.name || ''
      form.value.description = newData.description || ''
      form.value.unit_price = newData.unit_price || 0
      form.value.reorder_point = newData.reorder_point || 0
      form.value.safety_stock = newData.safety_stock || 0
      form.value.supplier_id = newData.supplier_id || ''
      form.value.category_id = newData.category_id || ''
    } else {
      handleReset()
    }
  },
  { immediate: true }
)

onMounted(async () => {
  await fetchSuppliers()
  await fetchCategory()
})

const isSubmitDisabled = computed(() => {
  return !(
    form.value.sparepart_id &&
    form.value.name &&
    form.value.description &&
    form.value.category_id &&
    form.value.supplier_id
  )
})
</script>
