<template>
  <div class="banner">
    <h1>Control User</h1>
  </div>
  <div class="wrapper">
    <a-flex justify="center" align="center">
      <UserList @promote="handlePromote" @demote="handleDemote" />
    </a-flex>
  </div>
</template>

<script setup>
import UserList from './UserList.vue'
import { message } from 'ant-design-vue'

const handlePromote = () => {
  message.success('Success promote roles')
}

const handleDemote = () => {
  message.success('Success demote roles')
}
</script>

<style scoped>
.banner {
  text-align: center;
}
.banner p {
  font-size: 1rem;
}
.wrapper {
  padding-right: 5%;
  padding-left: 5%;
}
</style>
