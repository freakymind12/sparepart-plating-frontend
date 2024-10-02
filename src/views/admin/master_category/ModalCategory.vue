<template>
  <a-modal v-bind:open="props.visible" :footer="null" @cancel="emits('close')" centered>
    <h2>{{ capitalizeEachWord(props.mode + ' category') }}</h2>
    <a-form
      layout="vertical"
      :model="form"
      @finish="props.mode === 'add' ? handleAdd() : handleEdit(props.data.category_id)"
    >
      <a-form-item label="Category ID" name="category_id" :rules="[{ required: true }]">
        <a-input placeholder="Category ID" v-model:value="form.category_id" />
      </a-form-item>
      <a-form-item label="Description" name="description" :rules="[{ required: true }]">
        <a-input placeholder="Description" v-model:value="form.description" />
      </a-form-item>
      <a-form-item>
        <a-flex justify="flex-end">
          <a-space :size="10">
            <a-button
              type="primary"
              html-type="submit"
              :disabled="form.category_id === '' || form.description === ''"
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
  category_id: '',
  description: ''
})

function capitalizeEachWord(str) {
  return str
    .split(' ') // Split the string into an array of words
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase()) // Capitalize each word
    .join(' ') // Join the words back into a single string
}

const emits = defineEmits(['close', 'edit', 'add'])
const props = defineProps({
  visible: Boolean,
  data: Object,
  mode: String
})

const handleEdit = async (id) => {
  try {
    await axios.patch(`http://192.168.148.125:5003/api/category/${id}`, {
      category_id: form.value.category_id,
      description: form.value.description
    })
    emits('edit')
    emits('close')
    message.success('Success edit category data')
  } catch (error) {
    console.error(error)
  }
}

const handleAdd = async () => {
  try {
    await axios.post(`http://192.168.148.125:5003/api/category/`, {
      category_id: form.value.category_id,
      description: form.value.description
    })
    emits('add')
    emits('close')
    message.success('Success add category')
    handleReset()
  } catch (error) {
    console.error(error)
  }
}

const handleReset = () => {
  form.value = {
    category_id: '',
    description: ''
  }
}

watch(
  () => props.data,
  (newData) => {
    if (newData) {
      form.value.category_id = newData.category_id || ''
      form.value.description = newData.description || ''
    } else {
      handleReset()
    }
  },
  { immediate: true }
)
</script>
