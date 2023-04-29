<template>
  <div class="employeeView">
    <div
      class="employeeCard"
      v-for="employee in employees"
      :key="employee.id"
      @click="$emit('select-employee', employee.id)"
    >
      <div>
        <img src="/user-regular.svg" alt="user-image" class="userImage" />
      </div>
      <div>{{ employee.name }} {{ employee.surname }}</div>
      <div>{{ employee.email }}</div>
      <div>{{ employee.is_active ? 'Active' : 'Inactive' }}</div>
      <div>{{ employee.is_manager ? 'Manager' : '' }}</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { Employee } from '@/types/interfaces'
import type { PropType } from 'vue'
defineProps({
  employees: Object as PropType<Employee[]>
})

defineEmits(['select-employee'])
</script>

<style scoped>
.employeeView {
  display: flex;
  justify-content: center;
  gap: 2%;
}

.employeeCard {
  padding: 1%;
  border: 1px yellow solid;
  border-radius: 1rem;
  max-width: 20%;
  cursor: pointer;
}

.employeeCard div {
  display: flex;
  justify-content: center;
  padding: 3px;
  text-align: center;
}

.userImage {
  width: 50%;
  color: yellow;
}
</style>
