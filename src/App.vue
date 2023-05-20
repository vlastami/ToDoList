<script setup>
import { ref } from 'vue';
import TodosDataTable from './components/TodosDataTable.vue';

const newTodo = ref('');
const changes = ref(0);

const jsonHeaders = () => ({
  'Content-Type': 'application/json',
});

async function createTodo(todo) {
  const response = await fetch('http://localhost:8000/api/todos', {
    method: 'POST',
    headers: jsonHeaders(),
    body: JSON.stringify(todo)
  });

  await response.json();
}

async function addTodo() {
  await createTodo({
    name: newTodo.value
  })

  newTodo.value = '';

  // await loadTodos();
}
</script>

<template>
  <header>
    <h1>TodoList</h1>
  </header>

  <main>
    <form @submit.prevent="addTodo">
      <input
          v-model="newTodo"
          type="text"
          placeholder="E.g. Buy milk"
      >

      <button type="submit">Add item</button>
    </form>

    <h2>Důležité úkoly</h2>
    <TodosDataTable
        :priority="10"
        :changes="changes"
        @todos-changed="changes++"
    />

    <h2>Všechny úkoly</h2>
    <TodosDataTable
        :changes="changes"
        @todos-changed="changes++"
    />
  </main>
</template>
