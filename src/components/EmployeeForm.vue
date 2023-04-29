<template>
  <div class="employeeFormWindow" @click="$emit('close-form')">
    <!-- Form to add or edit an employee -->
    <form class="employeeFormDiv" @click.stop="" @submit.prevent="onSubmitForm">
      <div class="closeButton" @click="$emit('close-form')">X</div>
      <div>
        <label for="name"> First Name*: </label> <br />
        <input class="formInput" type="text" name="name" id="name" v-model="formInput.name" />
        <br />

        <label for="surname"> Last Name*: </label> <br />
        <input
          class="formInput"
          type="text"
          name="surname"
          id="surname"
          v-model="formInput.surname"
        /><br />
        <label for="email">Email*: </label> <br />
        <input class="formInput" type="text" name="email" id="email" v-model="formInput.email" />
        <br />
        <label for="manager">Manager: </label> <br />
        <select class="formInput" name="manager" id="manager" v-model="formInput.manager">
          <option value="0">No manager</option>
          <option v-for="manager in managers" :key="manager.id" :value="manager.id">
            {{ manager.name }}
          </option>
        </select>
        <br />
        <label for="isActive">Is Active:</label
        ><input
          class="formInput"
          type="checkbox"
          id="isActive"
          name="isActive"
          v-model="formInput.isActive"
        />
        <br />
        <label for="isManager">Is Manager:</label
        ><input
          class="formInput"
          type="checkbox"
          id="isManager"
          name="isManager"
          v-model="formInput.isManager"
          :disabled="managerTeam?.length ? true : false"
        />
        <br />

        <div class="actionButtons">
          <button class="submitForm" @click.prevent="onSubmitForm">Save</button>
          <button
            v-if="selectedEmployee && !managerTeam?.length"
            class="submitForm"
            @click.prevent="onDelete"
          >
            Delete
          </button>
        </div>
        <div v-if="isError" class="errorMessage">
          <!-- Error feedback to the user, in case the form is not well written or there is an error while fetching -->
          {{ errorMessage }}
        </div>
        <div v-if="errorFetching" class="errorMessage">
          {{ errorMessage }}
        </div>
      </div>
    </form>
    <div v-if="selectedEmployee && managerTeam?.length" class="managerView">
      <!-- List of employees reporting to the manager -->
      <h2>Members of {{ selectedEmployee.name }}'s team:</h2>
      <div v-for="member in managerTeam" :key="member.id">{{ member.name }}</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import type { PropType } from 'vue'
import * as Yup from 'yup'
import {} from 'vue'
import type { Employee } from '@/types/interfaces'
import { useFetch } from '@vueuse/core'

const props = defineProps({
  managers: Object as PropType<{ id: number; name: string }[]>,
  selectedEmployee: Object as PropType<Employee | null>,
  managerTeam: Object as PropType<Employee[] | null>
})

const emit = defineEmits(['close-form'])

// Validation for the form, also loading values if employee is being edited
const formInput = reactive({
  name: props.selectedEmployee ? props.selectedEmployee.name : '',
  surname: props.selectedEmployee ? props.selectedEmployee.surname : '',
  email: props.selectedEmployee ? props.selectedEmployee.email : '',
  manager: props.selectedEmployee?.manager ? props.selectedEmployee.manager : 0,
  isActive: props.selectedEmployee ? props.selectedEmployee.is_active : false,
  isManager: props.selectedEmployee ? props.selectedEmployee.is_manager : false
})

const schema = Yup.object({
  name: Yup.string().required(),
  surname: Yup.string().required(),
  email: Yup.string()
    .email('Invalid email format')
    .matches(
      /^(([^<>()[\]\\.,;:\s@"]+(\.[^<>()[\]\\.,;:\s@"]+)*)|(".+"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/,
      'Email is invalid'
    )
    .required(),
  manager: Yup.number(),
  isActive: Yup.boolean(),
  isManager: Yup.boolean()
})

const isError = ref(false)
const errorFetching = ref(false)
const errorMessage = ref('')

// URL to make the request depending on if it's adding (POST) or editing (PUT) an employee
const url = props.selectedEmployee
  ? `http://127.0.0.1:8000/employee/${props.selectedEmployee.id}/`
  : 'http://127.0.0.1:8000/employee/'

async function addEmployee(employee: Employee) {
  await useFetch(url, {
    method: props.selectedEmployee ? 'PUT' : 'POST',
    headers: {
      Accept: 'application/json',
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(employee)
  })

  emit('close-form')
}

function onSubmitForm() {
  try {
    schema.validateSync(formInput)

    const employee = {
      id: props.selectedEmployee ? props.selectedEmployee.id : null,
      name: formInput.name,
      surname: formInput.surname,
      email: formInput.email,
      manager: formInput.manager != 0 ? formInput.manager : null,
      is_active: formInput.isActive,
      is_manager: formInput.isManager
    } as Employee
    addEmployee(employee)
  } catch (error) {
    console.error(error)
    isError.value = true
    errorMessage.value = error as string
  }
}

async function onDelete() {
  await useFetch(url, {
    method: 'DELETE',
    headers: {
      Accept: 'application/json',
      'Content-Type': 'application/json'
    }
  })

  emit('close-form')
}
</script>

<style>
.employeeFormWindow {
  position: fixed;
  padding-top: 5%;
  margin: 0;

  top: 0;
  left: 0;

  width: 100%;
  height: 100%;
  background-color: #000;
  z-index: 10;

  display: flex;
  justify-content: center;
  gap: 5%;
}

.closeButton {
  position: absolute;
  top: 20px;
  right: 20px;
  color: black;
  font-weight: bold;
  cursor: pointer;
}

.employeeFormDiv {
  display: flex;
  justify-content: center;
  max-width: 33%;
  background-color: rgba(255, 251, 0, 0.9);
  border-radius: 1rem;
  height: fit-content;

  padding: 50px;
  z-index: 20;
}

label {
  font-family: sans-serif;
  letter-spacing: 6px;
  text-transform: uppercase;
  font-size: 0.8em;
  color: #000;
}

.formInput[type='text'],
select {
  display: inline-block;
  border: none;
  text-align: left;
  box-shadow: 3px 2px rgba(121, 83, 210, 0.3);
  padding: 10px;
  width: 350px;
  margin: 10px 0;
  background: transparent;
  color: #000;
}

.formInput[type='text']:focus {
  background: none;
  border: none;
  color: #000;
}

.submitForm {
  background: transparent;
  color: #000;
  font-family: sans-serif;
  text-transform: uppercase;
  letter-spacing: 3px;
  margin: 20px 0;
  padding: 10px 30px;
  border: 2px solid #000;
}

.submitForm:hover {
  background: transparent;
  border: 2px solid rgba(69, 39, 160, 0.4);
}

.errorMessage {
  color: red;
}

.actionButtons {
  display: flex;
  justify-content: center;
  gap: 20%;
}

.managerView {
  display: flex;
  flex-direction: column;
  padding: 5%;
  background-color: rgba(255, 251, 0, 0.9);
  color: black;
  align-items: center;
  border-radius: 1rem;
  height: fit-content;
  min-height: 510px;
}
</style>
