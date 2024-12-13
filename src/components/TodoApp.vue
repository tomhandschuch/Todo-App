<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-400 to-indigo-600 flex items-center justify-center p-4">
    <div class="bg-white rounded-lg shadow-xl w-full max-w-md overflow-hidden">
      <h1 class="text-3xl font-bold text-center py-6 bg-gradient-to-r blue-500 to-indigo-600 text-white">
        Todo App
      </h1>
      <div class="p-6">
        <form @submit.prevent="addTodo" class="mb-6">
          <div class="flex items-center border-b border-blue-500 py-2">
            <input
              v-model="newTodo"
              class="appearance-none bg-transparent border-none w-full text-gray-700 mr-3 py-1 px-2 leading-tight focus:outline-none"
              type="text"
              placeholder="Add a new todo"
              aria-label="Add a new todo"
            >
            <button
              class="flex-shrink-0 bg-blue-500 hover:bg-blue-700 border-blue-500 hover:border-blue-700 text-sm border-4 text-white py-1 px-2 rounded"
              type="submit"
            >
              <PlusIcon class="h-5 w-5" />
            </button>
          </div>
          <p v-if="error" class="text-red-500 text-xs italic mt-2">{{ error }}</p>
        </form>
        
        <TransitionGroup name="list" tag="ul" class="space-y-2">
          <li
            v-for="todo in todos"
            :key="todo.id"
            class="flex items-center bg-gray-100 p-4 rounded-lg shadow transition duration-500 ease-in-out hover:bg-gray-200"
          >
            <input
              :id="'todo-' + todo.id"
              type="checkbox"
              :checked="todo.completed"
              @change="toggleTodo(todo)"
              class="form-checkbox h-5 w-5 text-blue-600 transition duration-150 ease-in-out"
            >
            <label
              :for="'todo-' + todo.id"
              :class="['ml-3 block text-gray-900 leading-5 w-full', { 'line-through text-gray-500': todo.completed }]"
              @dblclick="startEditing(todo)"
            >
              <span v-if="!todo.editing">{{ todo.text }}</span>
              <input
                v-else
                v-model="todo.text"
                @blur="finishEditing(todo)"
                @keyup.enter="finishEditing(todo)"
                @keyup.esc="cancelEditing(todo)"
                class="bg-white border border-gray-300 rounded px-2 py-1 w-full"
                ref="editInput"
              >
            </label>
            <button
              @click="removeTodo(todo)"
              class="ml-2 text-red-600 hover:text-red-800 transition duration-150 ease-in-out"
            >
              <TrashIcon class="h-5 w-5" />
            </button>
          </li>
        </TransitionGroup>
        
        <div v-if="todos.length === 0" class="text-center text-gray-500 mt-6">
          No todos yet. Add one above!
        </div>
        
        <div v-if="todos.length > 0" class="mt-6 flex justify-between items-center text-sm text-gray-600">
          <span>{{ activeTodosCount }} items left</span>
          <button
            @click="clearCompleted"
            class="text-blue-600 hover:text-blue-800 transition duration-150 ease-in-out"
          >
            Clear completed
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { PlusIcon, TrashIcon } from 'lucide-vue-next'

const todos = ref([])
const newTodo = ref('')
const error = ref('')

const addTodo = () => {
  if (newTodo.value.trim()) {
    todos.value.push({
      id: Date.now(),
      text: newTodo.value.trim(),
      completed: false,
      editing: false
    })
    newTodo.value = ''
    error.value = ''
  } else {
    error.value = 'Todo cannot be empty'
  }
}

const removeTodo = (todo) => {
  todos.value = todos.value.filter(t => t.id !== todo.id)
}

const toggleTodo = (todo) => {
  todo.completed = !todo.completed
}

const startEditing = (todo) => {
  todo.editing = true
  todo.originalText = todo.text
  setTimeout(() => {
    const input = document.querySelector(`input[type="text"][value="${todo.text}"]`)
    if (input) input.focus()
  }, 0)
}

const finishEditing = (todo) => {
  if (todo.text.trim()) {
    todo.editing = false
  } else {
    removeTodo(todo)
  }
}

const cancelEditing = (todo) => {
  todo.text = todo.originalText
  todo.editing = false
}

const clearCompleted = () => {
  todos.value = todos.value.filter(todo => !todo.completed)
}

const activeTodosCount = computed(() => {
  return todos.value.filter(todo => !todo.completed).length
})
</script>

<style scoped>
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
</style>