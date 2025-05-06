<template>

  <h1 class="app-title">ToDo-App</h1>

  <!-- Neue Aufgabe. -->
  <form @submit.prevent="createTodo" class="todo-neu">
    <input
      v-model="todoTitel"
      placeholder="Titel"
      required
    />
    <textarea
      v-model="todoBeschreibung"
      placeholder="Beschreibung..."
    ></textarea>
    <button type="submit">Speichern</button>
  </form>


  <!-- Bestehende ToDo einzeigen. -->
  <ul class="todo-liste">
    <li v-for="todo in todos" :key="todo.id" class="todo-button">
      <div v-if="editingId === todo.id" class="todo-change">

        <input v-model="todoChangeTitle" />
        <textarea v-model="todoChangeDescription"></textarea>

        <label>
          <input type="checkbox" v-model="completedChange" />
          Erledigt
        </label>

        
        <button @click="changeValues(todo.id)">Aktualisieren</button>
        <button @click="stopChange">Abbrechen</button>

      </div>

      <div v-else class="todo-display">

        <input
          type="checkbox"
          :checked="todo.completed"
          @change="togglePush(todo)"
        />

        <div class="todo-basic">
          <strong :class="{ done: todo.completed }">{{ todo.title }}</strong>
          <p class="description">{{ todo.description }}</p>
        </div>

        <button @click="todoEdit(todo)">Bearbeiten</button>
        <button @click="todoDelete(todo.id)">Löschen</button>

      </div>
    </li>
  </ul>
</template>








<script setup lang="ts">
import { ref } from 'vue'

const config = useRuntimeConfig()
const { data: todos, refresh } = await useFetch(
  `${config.public.apiBase}/todos/`
)

// erstellen
const todoTitel = ref('')
const todoBeschreibung = ref('')
const newCompleted = ref(false)

// bearbeiten
const editingId = ref<number | null>(null)
const todoChangeTitle = ref('')
const todoChangeDescription = ref('')
const completedChange = ref(false)

// posten
async function createTodo() {
  await $fetch(`${config.public.apiBase}/todos/`, {
    method: 'POST',
    body: {
      title: todoTitel.value.trim(),
      description: todoBeschreibung.value.trim(),
      completed: newCompleted.value
    }
  })
  todoTitel.value = ''
  todoBeschreibung.value = ''
  newCompleted.value = false
  await refresh()
}

// löschen
async function todoDelete(id: number) {
  await $fetch(`${config.public.apiBase}/todos/${id}/`, {
    method: 'DELETE'
  })
  await refresh()
}

// erledigt toggle
async function togglePush(item) {
  await $fetch(`${config.public.apiBase}/todos/${item.id}/`, {
    method: 'PUT',
    body: {
      title: item.title,
      description: item.description,
      completed: !item.completed
    }
  })
  await refresh()
}

// bearbeiten
function todoEdit(todo) {
  editingId.value = todo.id
  todoChangeTitle.value = todo.title
  todoChangeDescription.value = todo.description
  completedChange.value = todo.completed
}

// bearbeiten abbrechen
function stopChange() {
  editingId.value = null
}

// Bearbeitung abschließen
async function changeValues(id: number) {
  await $fetch(`${config.public.apiBase}/todos/${id}/`, {
    method: 'PUT',
    body: {
      title: todoChangeTitle.value.trim(),
      description: todoChangeDescription.value.trim(),
      completed: completedChange.value
    }
  })
  editingId.value = null
  await refresh()
}
</script>

<style scoped src="./todos.css"></style>