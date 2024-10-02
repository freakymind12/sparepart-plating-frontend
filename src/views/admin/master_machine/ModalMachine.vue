<template>
  <a-modal v-bind:open="props.visible" :footer="null" @cancel="emits('close')" centered>
    <h2>{{ capitalizeEachWord(props.mode + ' machine') }}</h2>
    <a-form
      layout="vertical"
      :model="form"
      @finish="props.mode === 'add' ? handleAdd() : handleEdit(props.data.machine_id)"
    >
      <a-form-item label="Machine ID" name="machine_id" :rules="[{ required: true }]">
        <a-input placeholder="Machine ID" v-model:value="form.machine_id" />
      </a-form-item>
      <a-form-item label="Name" name="machine_name" :rules="[{ required: true }]">
        <a-input placeholder="Name" v-model:value="form.machine_name" />
      </a-form-item>
      <a-form-item>
        <a-flex justify="flex-end">
          <a-space :size="10">
            <a-button
              type="primary"
              html-type="submit"
              :disabled="form.machine_id === '' || form.machine_name === ''"
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
  machine_id: '',
  machine_name: ''
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
    await axios.patch(`http://192.168.148.125:5003/api/machine/${id}`, {
      machine_id: form.value.machine_id,
      machine_name: form.value.machine_name
    })
    emits('edit')
    emits('close')
    message.success('Success edit machine')
  } catch (error) {
    console.error(error)
  }
}

const handleAdd = async () => {
  try {
    await axios.post(`http://192.168.148.125:5003/api/machine/`, {
      machine_id: form.value.machine_id,
      machine_name: form.value.machine_name
    })
    emits('add')
    emits('close')
    message.success('Success add machine')
    handleReset()
  } catch (error) {
    console.error(error)
  }
}

const handleReset = () => {
  form.value = {
    machine_id: '',
    machine_name: ''
  }
}

watch(
  () => props.data,
  (newData) => {
    if (newData) {
      form.value.machine_id = newData.machine_id || ''
      form.value.machine_name = newData.machine_name || ''
    } else {
      handleReset()
    }
  },
  { immediate: true }
)
</script>
