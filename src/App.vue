<template>
  <v-app>
    <v-app-bar color="primary" dark>
      <v-app-bar-title>📝 Task Manager</v-app-bar-title>
    </v-app-bar>

    <v-container class="mt-5">
      <!-- Add Task Form -->
      <v-card class="mb-5">
        <v-card-title>Add New Task</v-card-title>
        
        <v-card-text>
          <v-row>
            <v-col cols="12" md="8">
              <v-text-field 
                v-model="newTask"
                label="Task name"
                placeholder="Enter your task"
                outlined
                @keyup.enter="addTask"
              />
            </v-col>
            <v-col cols="12" md="4">
              <v-select 
                v-model="newPriority"
                :items="['High', 'Medium', 'Low']"
                label="Priority"
                outlined
              />
            </v-col>
          </v-row>

          <v-btn 
            @click="addTask"
            color="success"
            size="large"
            block
          >
            + Add Task
          </v-btn>
        </v-card-text>
      </v-card>

      <!-- Statistics -->
      <v-row class="mb-5">
        <v-col cols="12" md="4">
          <v-card color="info">
            <v-card-text class="text-center">
              <p class="text-h6">Total Tasks</p>
              <p class="text-h4">{{ tasks.length }}</p>
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" md="4">
          <v-card color="warning">
            <v-card-text class="text-center">
              <p class="text-h6">Pending</p>
              <p class="text-h4">{{ pendingCount }}</p>
            </v-card-text>
          </v-card>
        </v-col>

        <v-col cols="12" md="4">
          <v-card color="success">
            <v-card-text class="text-center">
              <p class="text-h6">Completed</p>
              <p class="text-h4">{{ completedCount }}</p>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>

      <!-- Filter Buttons -->
      <v-row class="mb-5">
        <v-col cols="12">
          <v-btn-group variant="outlined" divided>
            <v-btn 
              @click="filter = 'all'"
              :color="filter === 'all' ? 'primary' : ''"
              :variant="filter === 'all' ? 'flat' : 'outlined'"
            >
              All
            </v-btn>
            <v-btn 
              @click="filter = 'pending'"
              :color="filter === 'pending' ? 'primary' : ''"
              :variant="filter === 'pending' ? 'flat' : 'outlined'"
            >
              Pending
            </v-btn>
            <v-btn 
              @click="filter = 'completed'"
              :color="filter === 'completed' ? 'primary' : ''"
              :variant="filter === 'completed' ? 'flat' : 'outlined'"
            >
              Completed
            </v-btn>
          </v-btn-group>
        </v-col>
      </v-row>

      <!-- Task List -->
      <v-card>
        <v-card-title>Tasks ({{ filteredTasks.length }})</v-card-title>

        <v-card-text>
          <!-- Empty State -->
          <v-empty-state
            v-if="filteredTasks.length === 0"
            headline="No tasks"
            text="Add a task to get started!"
          />

          <!-- Task Table -->
          <v-table v-else>
            <thead>
              <tr>
                <th class="text-left">Done</th>
                <th class="text-left">Task Name</th>
                <th class="text-left">Priority</th>
                <th class="text-left">Status</th>
                <th class="text-left">Action</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="task in filteredTasks" :key="task.id">
                <!-- Checkbox -->
                <td>
                  <v-checkbox
                    :model-value="task.completed"
                    @update:model-value="toggleTask(task.id)"
                    hide-details
                  />
                </td>

                <!-- Task Name -->
                <td>
                  <span :style="{ textDecoration: task.completed ? 'line-through' : 'none' }">
                    {{ task.name }}
                  </span>
                </td>

                <!-- Priority Badge -->
                <td>
                  <v-chip 
                    :color="getPriorityColor(task.priority)"
                    text-color="white"
                    small
                  >
                    {{ task.priority }}
                  </v-chip>
                </td>

                <!-- Status Badge -->
                <td>
                  <v-chip 
                    :color="task.completed ? 'success' : 'warning'"
                    text-color="white"
                    small
                  >
                    {{ task.completed ? 'Completed' : 'Pending' }}
                  </v-chip>
                </td>

                <!-- Delete Button -->
                <td>
                  <v-btn 
                    @click="deleteTask(task.id)"
                    color="error"
                    size="small"
                    variant="tonal"
                  >
                    Delete
                  </v-btn>
                </td>
              </tr>
            </tbody>
          </v-table>
        </v-card-text>
      </v-card>
    </v-container>
  </v-app>
</template>

<script setup>
import { ref, computed } from 'vue'

const newTask = ref('')
const newPriority = ref('Medium')
const tasks = ref([])
const filter = ref('all')
let nextId = 1

// Add new task
function addTask() {
  if (newTask.value.trim() === '') {
    alert('Please enter a task name')
    return
  }

  tasks.value.push({
    id: nextId++,
    name: newTask.value,
    priority: newPriority.value,
    completed: false
  })

  newTask.value = ''
  newPriority.value = 'Medium'
}

// Toggle task completion
function toggleTask(id) {
  const task = tasks.value.find(t => t.id === id)
  if (task) {
    task.completed = !task.completed
  }
}

// Delete task
function deleteTask(id) {
  tasks.value = tasks.value.filter(t => t.id !== id)
}

// Get priority color
function getPriorityColor(priority) {
  const colors = {
    'High': 'error',
    'Medium': 'warning',
    'Low': 'success'
  }
  return colors[priority]
}

// Filtered tasks
const filteredTasks = computed(() => {
  if (filter.value === 'completed') {
    return tasks.value.filter(t => t.completed)
  } else if (filter.value === 'pending') {
    return tasks.value.filter(t => !t.completed)
  }
  return tasks.value
})

// Statistics
const completedCount = computed(() => {
  return tasks.value.filter(t => t.completed).length
})

const pendingCount = computed(() => {
  return tasks.value.filter(t => !t.completed).length
})
</script>