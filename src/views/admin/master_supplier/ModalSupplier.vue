<template>
  <a-modal v-bind:open="props.visible" :footer="null" @cancel="emits('close')" centered>
    <h2>{{ capitalizeEachWord(props.mode + ' supplier') }}</h2>
    <a-form
      layout="vertical"
      :model="form"
      @finish="props.mode === 'add' ? handleAdd() : handleEdit(props.data.supplier_id)"
    >
      <a-form-item label="Supplier Name" name="supplier_name" :rules="[{ required: true }]">
        <a-input placeholder="Supplier Name" v-model:value="form.supplier_name" />
      </a-form-item>
      <a-form-item label="Location" name="location" :rules="[{ required: true }]">
        <a-select v-model:value="form.location" placeholder="Location">
          <a-select-option value="Japan">Japan</a-select-option>
          <a-select-option value="Indonesia">Indonesia</a-select-option>
          <a-select-option value="Other">Other</a-select-option>
        </a-select>
      </a-form-item>
      <a-form-item>
        <a-flex justify="flex-end">
          <a-space :size="10">
            <a-button
              type="primary"
              html-type="submit"
              :disabled="form.supplier_name === '' || form.location === ''"
            >
              Submit
            </a-button>
          </a-space>
        </a-flex>
      </a-form-item>
    </a-form>
  </a-modal>
</template>

<script setup>
import { ref, watch } from 'vue'
import axios from 'axios'
import { message } from 'ant-design-vue'

const form = ref({
  supplier_name: '',
  location: 'Japan'
})

function capitalizeEachWord(str) {
  return str
    .split(' ')
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
    .join(' ')
}

const emits = defineEmits(['close', 'edit', 'add'])
const props = defineProps({
  visible: Boolean,
  data: Object,
  mode: String
})

const handleEdit = async (id) => {
  try {
    await axios.patch(`http://192.168.148.125:5003/api/supplier/${id}`, {
      supplier_name: form.value.supplier_name,
      location: form.value.location
    })
    emits('edit')
    emits('close')
    message.success('Success edit supplier')
  } catch (error) {
    console.error(error)
  }
}

const handleAdd = async () => {
  try {
    await axios.post(`http://192.168.148.125:5003/api/supplier/`, {
      supplier_name: form.value.supplier_name,
      location: form.value.location
    })
    emits('add')
    emits('close')
    message.success('Success add supplier')
    handleReset()
  } catch (error) {
    console.error(error)
  }
}

const handleReset = () => {
  form.value = {
    supplier_name: '',
    location: ''
  }
}

watch(
  () => props.data,
  (newData) => {
    if (newData) {
      form.value.supplier_name = newData.supplier_name || ''
      form.value.location = newData.location || ''
    } else {
      handleReset()
    }
  },
  { immediate: true }
)
</script>
