<template>
  <div class="home">
    <header class="header">
      <button class="addButton" @click="onAddEmployee">Add employee</button>
    </header>
    <main class="main">
      <div v-if="employees">
        <EmployeeTable :employees="employees" @select-employee="onSelectEmployee" />
      </div>
      <EmployeeForm
        v-if="isEmployeeFormOpen"
        :managers="managers"
        :selectedEmployee="selectedEmployee"
        :managerTeam="managerTeam"
        @close-form="onCloseForm"
      />
    </main>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import EmployeeTable from '../components/EmployeeTable.vue'
import EmployeeForm from '../components/EmployeeForm.vue'
import type { Employee } from '../types/interfaces'
import { useFetch } from '@vueuse/core'

const isEmployeeFormOpen = ref(false)
const selectedEmployee = ref<Employee | null>(null)
const managerTeam = ref<Employee[] | null>(null)

// Fetch the employees from the backend
const { data: employees } = await useFetch<Employee[]>('http://127.0.0.1:8000/employee/', {
  headers: {
    Accept: 'application/json',
    'Content-Type': 'application/json'
  }
})
  .get()
  .json()

// Get the managers from the array of employees
const managers = computed(() => {
  if (employees.value) {
    return employees.value.filter((employee: Employee) => {
      if (employee.is_manager) return { id: employee.id, name: employee.name }
    })
  }
  return []
})

// Open the form and delete any possible value previously loaded (when editing an employee i.e.)
function onAddEmployee() {
  isEmployeeFormOpen.value = true
  selectedEmployee.value = null
}

// Get the employees that are reporting with the employee selected
function getManagerTeam(managerId: number) {
  return employees.value.filter((e: Employee) => {
    if (e.manager === managerId) return e
  })
}

// Open the form to edit the employee selected
function onSelectEmployee(employeeId: number) {
  selectedEmployee.value = employees.value.find((e: Employee) => e.id == employeeId)
  managerTeam.value = getManagerTeam(employeeId)
  isEmployeeFormOpen.value = true
}

function onCloseForm() {
  location.reload()
}
</script>

<style scoped>
.home {
  padding: 0 5%;
}
.header {
  display: flex;
  justify-content: end;
}

.addButton {
  border: 1px yellow solid;
  padding: 10px;
  margin: 10px 0;
  background: transparent;
  color: yellow;
  font-size: large;
}
</style>
