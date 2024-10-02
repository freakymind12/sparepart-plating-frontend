<template>
  <div class="banner">
    <h1>
      <ArrowDownOutlined style="color: green" /> Incoming / Outgoing
      <ArrowUpOutlined style="color: red" />
    </h1>
  </div>
  <div class="wrapper">
    <a-flex align="center" justify="center" gap="large" wrap="wrap">
      <MasterIncoming />
      <MasterOutgoing />
    </a-flex>
  </div>
</template>

<script setup>
import { ArrowUpOutlined, ArrowDownOutlined } from '@ant-design/icons-vue'
import MasterIncoming from './incoming/MasterIncoming.vue'
import MasterOutgoing from './outgoing/MasterOutgoing.vue'
</script>

<style scoped>
.banner {
  text-align: center;
}

.wrapper {
  padding-right: 5%;
  padding-left: 5%;
  padding-top:2%
}
</style>
