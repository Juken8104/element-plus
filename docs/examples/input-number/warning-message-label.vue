<template>
  <el-input-number
    v-model="num"
    :min="1"
    :max="100"
    :value-on-clear="20"
    warning-message-label="数值"
  />
</template>

<script lang="ts" setup>
import { ref } from 'vue'

const num = ref(5)
</script>
