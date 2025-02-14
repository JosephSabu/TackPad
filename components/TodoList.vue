<template>
  <div class="h-full flex flex-col bg-white rounded-lg">
    <div class="p-4 border-b flex items-center justify-between">
      <div class="flex items-center gap-2">
        <input
          v-model="localTitle"
          class="text-xl font-semibold bg-transparent focus:outline-none focus:ring-2 focus:ring-blue-500 rounded px-1"
          @blur="updateTitle"
          @mousedown.stop
          @keydown.delete.stop
        />
      </div>
    </div>
    
    <div class="p-4 pt-0 overflow-auto" style="max-height: calc(100% - 64px);">
      <div class="bg-gray-50 rounded-lg mb-4 flex items-center">
        <input
          type="text"
          placeholder="Add a new task"
          class="w-full p-4 bg-gray-50 focus:outline-none text-gray-600"
          v-model="newTask"
          @keyup.enter="addNewTask"
          @mousedown.stop
          @keydown.delete.stop
        />
        <button 
          @click.stop="addNewTask"
          class="p-4 text-blue-600 hover:text-blue-800"
        >
          <span class="text-3xl">+</span>
        </button>
      </div>

      <ul class="space-y-4 overflow-y-auto">
        <li 
          v-for="task in list.content.tasks" 
          :key="task.task_id" 
          class="flex items-center gap-3"
        >
          <button 
            class="w-6 h-6 rounded border-2 border-blue-600 flex items-center justify-center"
            :class="{ 'bg-blue-600': task.completed }"
            @click.stop="toggleTask(task)"
          >
            <svg
              v-if="task.completed"
              class="w-4 h-4 text-white"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M5 13l4 4L19 7"
              />
            </svg>
          </button>
          <input
            v-if="editingTaskId === task.task_id"
            type="text"
            v-model="editingContent"
            class="flex-grow bg-transparent rounded px-1 focus:outline-none"
            @blur="saveTaskEdit(task)"
            @keyup.enter="saveTaskEdit(task)"
            @keyup.esc="cancelTaskEdit"
            @mousedown.stop
            ref="editInput"
          />
          <span 
            v-else
            class="flex-grow cursor-pointer"
            :class="{ 'line-through text-gray-400': task.completed }"
            @dblclick.stop="startEditing(task)"
          >
            {{ task.content }}
          </span>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, nextTick } from 'vue'
import { useBoardStore } from '~/stores/board'
import type { TodoList, Task } from '~/types/board'

const props = defineProps<{
  list: TodoList
  isSelected: boolean
}>()

const boardStore = useBoardStore()
const localTitle = ref(props.list.content.title)
const newTask = ref('')
const editingTaskId = ref<string | null>(null)
const editingContent = ref('')
const editInput = ref<HTMLInputElement | null>(null)

function updateTitle() {
  if (localTitle.value === props.list.content.title) return
  boardStore.updateItem(props.list.id, {
    content: {
      ...props.list.content,
      title: localTitle.value
    }
  })
}

function addNewTask() {
  if (!newTask.value.trim()) return
  boardStore.addTask(props.list.id, newTask.value)
  newTask.value = ''
}

function toggleTask(task: Task) {
  task.completed = !task.completed
  boardStore.updateItem(props.list.id, {
    content: {
      ...props.list.content,
      tasks: [...props.list.content.tasks]
    }
  })
}

function startEditing(task: Task) {
  editingTaskId.value = task.task_id
  editingContent.value = task.content
  nextTick(()=> {
    editInput.value[0].focus()
  })
  
}

function saveTaskEdit(task: Task) {
  if (editingTaskId.value === null) return
  if (editingContent.value.trim() !== '') {
    boardStore.updateTask(props.list.id, task.task_id, editingContent.value)
  }
  editingTaskId.value = null
  editingContent.value = ''
}

function cancelTaskEdit() {
  editingTaskId.value = null
  editingContent.value = ''
}
</script>

<style scoped>
.todo-list {
  min-width: 300px;
  min-height: 200px;
}

/* Custom scrollbar styles */
.overflow-auto::-webkit-scrollbar {
  width: 8px;
}

.overflow-auto::-webkit-scrollbar-track {
  background: transparent;
}

.overflow-auto::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 4px;
}

.overflow-auto::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}
</style>