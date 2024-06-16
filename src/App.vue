<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const todos = ref([])
const name = ref('') 
const input_content = ref('')
const undone = ref(false)

const todos_asc = computed(() => todos.value.sort((a, b) => {
  return b.createdAt - a.createdAt
}))

const addTodo = () => {
  if (input_content.value.trim() === '') {
    return
  }

  todos.value.push({
    content: input_content.value,
    done: false,
    createdAt: new Date().getTime(),
    isEditing: false
  })

  input_content.value = ''
}

const filteredTodos = computed(() => {
  if (undone.value) {
    return todos_asc.value.filter(todo => !todo.done)
  }
  return todos_asc.value
})

const toggleundone = () => {
  undone.value = !undone.value 
}

const deleteAllTodos = () => {
  todos.value = [];
}

const deletedTodos = ref([]); 
const deleteAll = () => {
  deletedTodos.value = [...todos.value];
  todos.value = [];
}

const removeTodo = todo => {
  todos.value = todos.value.filter(t => t !== todo)
}

const toggleEdit = (todo) => {
  todo.isEditing = !todo.isEditing
}

const filterTodos = (status) => {
  // Pastikan untuk memperbarui nilai name dengan status yang diberikan
  name.value = status
}

watch(todos, newVal => {
  localStorage.setItem('todos', JSON.stringify(newVal))
}, { deep: true })

watch(name, (newVal) => {
  localStorage.setItem('name', newVal)
})

onMounted(() => {
  name.value = localStorage.getItem('name') || 'all'
  todos.value = JSON.parse(localStorage.getItem('todos')) || []
})
</script>


<template>
<main class="app">
  <section class="greeting" style="text-align: center;">
    <h2 class="title" style="display: inline-block; width: 100%;">
      Haii.. What Are You Want To Do ??
    </h2>
  </section>

  <section class="create-todo">
    <h3>List Your Todo Here</h3>

    <form @submit.prevent="addTodo">
      <input type="text" placeholder="Text Here" v-model="input_content" />
      <input type="submit" value="Add todo" />
    </form>
  </section>

  <section class="todo-list">
    <h3>TODO LIST</h3>
    <div class="list">
      <section class="filter">
        <button @click="toggleundone" class="undone" style="margin-right: 10px;">
          Undone
        </button>
        <button class="delete-all" @click="deleteAll" style="background-color: red;">Delete All</button>
      </section>
      <div v-for="todo in filteredTodos" :key="todo.createdAt" :class="`todo-item ${todo.done && 'done'}`">
  <label>
    <input type="checkbox" v-model="todo.done" />
    <span class="bubble"></span>
  </label>
  <div class="todo-content">
    <input v-if="!todo.isEditing" type="text" v-model="todo.content" disabled />
    <input v-else type="text" v-model="input_content" />
  </div>
  <div class="actions">
    <button v-if="!todo.isEditing" class="edit" @click="toggleEdit(todo)">Edit</button>
    <button v-else class="edit save" @click="toggleEdit(todo)">Save</button>
    <button class="delete" @click="removeTodo(todo)">Delete</button>
  </div>
</div>
    </div>
  </section>
</main>
</template>
