<template>
  <div class="banner">
    <h1>Master Data</h1>
  </div>
  <div class="wrapper">
    <a-flex align="center" gap="small" wrap="wrap">
      <MasterSparepart />
      <MasterCategory />
      <MasterSupplier />
      <MasterMachine />
    </a-flex>
  </div>
</template>

<script setup>
import MasterCategory from './master_category/MasterCategory.vue'
import MasterSparepart from './master_sparepart/MasterSparepart.vue'
import MasterSupplier from './master_supplier/MasterSupplier.vue'
import MasterMachine from './master_machine/MasterMachine.vue'
</script>

<style scoped>
.banner {
  text-align: center;
}

.wrapper {
  padding-right: 5%;
  padding-left: 5%;
}
</style>
