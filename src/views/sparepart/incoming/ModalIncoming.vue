<template>
  <a-modal v-bind:open="props.visible" :footer="null" @cancel="emits('close')" centered>
    <h2>{{ capitalizeEachWord(props.mode + ' incoming') }}</h2>
    <a-form
      layout="vertical"
      :model="form"
      @finish="props.mode === 'add' ? handleAdd() : handleEdit(props.data.goods_in_id)"
    >
      <a-form-item label="Sparepart" name="sparepart_id" :rules="[{ required: true }]">
        <a-select
          v-model:value="form.sparepart_id"
          placeholder="Sparepart"
          :options="sparepartList"
          show-search
          :filter-option="filterOption"
        >
        </a-select>
      </a-form-item>
      <a-row :gutter="16">
        <a-col :span="12">
          <a-form-item label="Qty" name="quantity_in" :rules="[{ required: true }]">
            <a-input-number
              :min="0"
              placeholder="Qty"
              v-model:value="form.quantity_in"
              style="width: 100%"
            />
          </a-form-item>
        </a-col>
        <a-col :span="12">
          <a-form-item label="Date" name="date_in" :rules="[{ required: true }]">
            <a-date-picker v-model:value="form.date_in" style="width: 100%" format="YYYY-MM-DD"/>
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
              :disabled="form.sparepart_id === '' || form.quantity_in === 0 || form.date_in === ''"
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
  quantity_in: 0,
  date_in: null,
  id_user: ''
})
const sparepartList = ref([])

const handleEdit = async (id) => {
  try {
    const date_in = convertUtcTimesToLocal(form.value.date_in)
    await axios.patch(`http://192.168.148.125:5003/api/sparepart-incoming/${id}`, {
      sparepart_id: form.value.sparepart_id,
      quantity_in: form.value.quantity_in,
      date_in: date_in
    })
    emits('edit')
    emits('close')
    message.success('Success edit incoming')
  } catch (error) {
    console.error(error)
  }
}

const handleAdd = async () => {
  try {
    const date_in = convertUtcTimesToLocal(form.value.date_in)
    await getUserData()
    console.log(form.value.id_user)
    await axios.post(`http://192.168.148.125:5003/api/sparepart-incoming/`, {
      sparepart_id: form.value.sparepart_id,
      quantity_in: form.value.quantity_in,
      date_in: date_in,
      id_user:form.value.id_user
    })
    emits('add')
    emits('close')
    message.success('Success add incoming')
    handleReset()
  } catch (error) {
    console.error(error)
  }
}

const handleReset = () => {
  form.value = {
    sparepart_id: '',
    quantity_in: 0,
    date_in: ''
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
  return option.value.toLowerCase().indexOf(input.toLowerCase()) >= 0;
};

const convertUtcTimesToLocal = (utcTimes) => {
  return dayjs(utcTimes).utc().tz('Asia/Jakarta').format('YYYY-MM-DD')
}

watch(
  () => props.data,
  (newData) => {
    if (newData) {
      form.value.sparepart_id = newData.sparepart_id || ''
      form.value.quantity_in = newData.quantity_in || 0
      form.value.date_in = newData.date_in ? dayjs(newData.date_in) : ''
    } else {
      handleReset()
    }
  },
  { immediate: true }
)

onMounted(async () => {
  await fetchSparepart()
})
</script>
