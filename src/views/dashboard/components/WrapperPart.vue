<template>
  <div class="card-wrapper">
    <a-flex justify="center" style="margin-bottom: 0.3rem">
      <h3 style="font-size: 1.2rem;">{{ props.title }}</h3>
    </a-flex>
    <a-flex vertical gap="middle">
      <slot name="sparepart-content" />
    </a-flex>
  </div>
</template>

<script setup>
const props = defineProps({
  title:String,
})
</script>

<style scoped>
.card-wrapper {
  min-width: 400px;
  max-width: 400px;
  min-height: 700px;
  border: 1px solid #333;
  padding: 1rem;
  border-radius: 10px;
}
</style>
