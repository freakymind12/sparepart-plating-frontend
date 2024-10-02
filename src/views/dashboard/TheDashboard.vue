<template>
  <div class="banner">
    <h1>Dashboard</h1>
  </div>
  <div class="wrapper">
    <a-flex justify="center" align="center" gap="small" wrap="wrap">
      <WrapperPart title="Machine Sparepart">
        <template #sparepart-content>
          <CardPart category="MACHINE"/>
        </template>
      </WrapperPart>
      <a-flex vertical gap="small">
        <WrapperWidget title="Machine" :refresh="refreshWidget" @refresh-completed="handleRefreshCompleted()"/>
        <WrapperWidget title="Lab" :refresh="refreshWidget"/>
      </a-flex>
      <WrapperPart title="Lab Sparepart">
        <template #sparepart-content>
          <CardPart category="LAB" @add="handleAddPart()"/>
        </template>
      </WrapperPart>
    </a-flex>
  </div>
</template>

<script setup>
import WrapperPart from './components/WrapperPart.vue'
import CardPart from './components/CardPart.vue';
import WrapperWidget from './components/WrapperWidget.vue';
import { ref } from 'vue';

const refreshWidget = ref(false)

const handleAddPart = () => {
  refreshWidget.value = true
}

const handleRefreshCompleted = () => {
  refreshWidget.value = false
}

</script>

<style scoped>
.banner {
  text-align: center;
}
.wrapper {
  padding-left: 5%;
  padding-right: 5%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
