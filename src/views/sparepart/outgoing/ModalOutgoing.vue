<template>
  <a-modal v-bind:open="props.visible" :footer="null" @cancel="emits('close')" centered>
    <h2>{{ capitalizeEachWord(props.mode + ' outgoing') }}</h2>
    <a-form
      layout="vertical"
      :model="form"
      @finish="props.mode === 'add' ? handleAdd() : handleEdit(props.data.goods_out_id)"
    >
      <a-form-item label="Sparepart" name="sparepart_id" :rules="[{ required: true }]">
        <a-select
          v-model:value="form.sparepart_id"
          placeholder="Sparepart"
          :options="sparepartList"
          show-search
          :filter-option="filterOption"
        />
      </a-form-item>
      <a-row :gutter="16">
        <a-col :span="12">
          <a-form-item label="Qty" name="quantity_out" :rules="[{ required: true }]">
            <a-input-number
              :min="0"
              placeholder="Qty"
              v-model:value="form.quantity_out"
              style="width: 100%"
            />
          </a-form-item>
        </a-col>
        <a-col :span="12">
          <a-form-item label="Machine" name="machine_id" :rules="[{ required: true }]">
            <a-select
              v-model:value="form.machine_id"
              placeholder="Machine"
              :options="machineList"
              show-search
              :filter-option="filterOption"
            />
          </a-form-item>
        </a-col>
      </a-row>
      <a-row :gutter="16">
        <a-col :span="12">
          <a-form-item label="Date" name="date_out" :rules="[{ required: true }]">
            <a-date-picker v-model:value="form.date_out" style="width: 100%" format="YYYY-MM-DD" />
          </a-form-item>
        </a-col>
        <a-col :span="12">
          <a-form-item label="Note" name="note" :rules="[{ required: true }]">
            <a-textarea v-model:value="form.note" placeholder="Note" auto-size />
          </a-form-item>
        </a-col>
      </a-row>

      <!-- Submit button -->
      <a-form-item>
        <a-flex justify="flex-end">
          <a-space :size="10">
            <a-button
              type="primary"
              html-type="submit"
              :disabled="
                form.sparepart_id === '' ||
                form.quantity_out === 0 ||
                form.date_out === '' ||
                form.note === ''
              "
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
import { onMounted, ref, watch } from 'vue'
import axios from 'axios'
import { message } from 'ant-design-vue'
import auth from '@/auth/auth'
import dayjs from 'dayjs'
import utc from 'dayjs/plugin/utc'
import timezone from 'dayjs/plugin/timezone'

dayjs.extend(utc)
dayjs.extend(timezone)

const emits = defineEmits(['close', 'edit', 'add'])
const props = defineProps({
  visible: Boolean,
  data: Object,
  mode: String
})
const form = ref({
  sparepart_id: '',
  quantity_out: 0,
  date_out: null,
  machine_id:'',
  note: '',
  id_user: ''
})
const sparepartList = ref([])
const machineList = ref([])

const handleEdit = async (id) => {
  try {
    const date_out = convertUtcTimesToLocal(form.value.date_out)
    await axios.patch(`http://192.168.148.125:5003/api/sparepart-outgoing/${id}`, {
      sparepart_id: form.value.sparepart_id,
      quantity_out: form.value.quantity_out,
      note: form.value.note,
      machine_id:form.value.machine_id,
      date_out: date_out
    })
    emits('edit')
    emits('close')
    message.success('Success edit outgoing')
  } catch (error) {
    console.error(error)
  }
}

const handleAdd = async () => {
  try {
    const date_out = convertUtcTimesToLocal(form.value.date_out)
    await getUserData()
    console.log(form.value.id_user)
    await axios.post(`http://192.168.148.125:5003/api/sparepart-outgoing/`, {
      sparepart_id: form.value.sparepart_id,
      quantity_out: form.value.quantity_out,
      date_out: date_out,
      machine_id: form.value.machine_id,
      note: form.value.note,
      id_user: form.value.id_user
    })
    emits('add')
    emits('close')
    message.success('Success add outgoing')
    handleReset()
  } catch (error) {
    console.error(error)
  }
}

const handleReset = () => {
  form.value = {
    sparepart_id: '',
    quantity_out: 0,
    date_out: '',
    machine_id:'',
    note: ''
  }
}

const fetchSparepart = async () => {
  try {
    const response = await axios.get('http://192.168.148.125:5003/api/sparepart')
    const data = response.data.data
    sparepartList.value = data.map((part) => ({
      value: part.sparepart_id,
      label: `[${part.sparepart_id}] - ${part.name}`
    }))
  } catch (error) {
    console.error(error)
  }
}

const fetchMachine = async () => {
  try {
    const response = await axios.get('http://192.168.148.125:5003/api/machine')
    const data = response.data.data
    machineList.value = data.map((machine) => ({
      value: machine.machine_id,
      label: machine.machine_id
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

const capitalizeEachWord = (str) => {
  return str
    .split(' ')
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
    .join(' ')
}

const filterOption = (input, option) => {
  return option.value.toLowerCase().indexOf(input.toLowerCase()) >= 0
}

const convertUtcTimesToLocal = (utcTimes) => {
  return dayjs(utcTimes).utc().tz('Asia/Jakarta').format('YYYY-MM-DD')
}

watch(
  () => props.data,
  (newData) => {
    if (newData) {
      form.value.sparepart_id = newData.sparepart_id || ''
      form.value.note = newData.note || ''
      form.value.machine_id = newData.machine_id || ''
      form.value.quantity_out = newData.quantity_out || 0
      form.value.date_out = newData.date_out ? dayjs(newData.date_out) : ''
    } else {
      handleReset()
    }
  },
  { immediate: true }
)

onMounted(async () => {
  await fetchSparepart()
  await fetchMachine()
})
</script>
