<template>
  <div :class="`alert alert-${type} alert-dismissible fade show`" role="alert">
    {{ message }}
    <button type="button" class="btn-close" @click="$emit('close')"></button>
  </div>
</template>

<script>
export default {
  name: 'NotificationMessage',
  props: {
    type: { 
      type: String, 
      default: 'success' 
    },
    message: { 
      type: String, 
      required: true 
    }
  }
}
</script>

<style scoped>
.alert {
  transition: opacity 0.5s ease;
}
</style>