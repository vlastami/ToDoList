<script setup>
import {ref, onMounted, reactive, watch} from 'vue';
import { format, parseISO } from 'date-fns';

const todos = reactive([]);
const newTodo = ref('');
const selectedSortOption = ref('default');

function jsonHeaders() {
  return {
    'Content-Type': 'application/json',
  };
}

watch(selectedSortOption, (newSortOption) => {
  switch (newSortOption) {
    case 'name-asc':
      todos.sort((a, b) => a.name.localeCompare(b.name));
      break;
    case 'name-desc':
      todos.sort((a, b) => b.name.localeCompare(a.name));
      break;
    case 'creation-asc':
      todos.sort((a, b) => parseISO(a.created_at) - parseISO(b.created_at));
      break;
    case 'creation-desc':
      todos.sort((a, b) => parseISO(b.created_at) - parseISO(a.created_at));
      break;
    default:
      break;
  }
});

async function loadTodos() {
  const response = await fetch('http://192.168.59.251:8000/api/todos?show_completed=1');
  const body = await response.json();

  todos.splice(0, todos.length, ...body.todos.map(todo => ({
    ...todo,
    created_at: format(parseISO(todo.created_at), 'dd. mm. yyyy')
  })));
}

async function createTodo(todo) {
  const response = await fetch('http://192.168.59.251:8000/api/todos', {
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

  await loadTodos();
}

async function toggleItem(todoId) {
  const todo = todos.value.find((todo) => todo.id === todoId);

  await fetch(`http://192.168.59.251:8000/api/todos/${todoId}`, {
    method: 'PUT',
    headers: jsonHeaders(),
    body: JSON.stringify({
      'completed_at': todo.completed_at === null
          ? format(new Date(), 'yyyy-MM-dd HH:mm:ss')
          : null
    })
  });

  await loadTodos();
}

onMounted(() => {
  loadTodos();
});
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

    <label for="hide-completed">
      <input
          type="checkbox"
          id="hide-completed"
      >

      Hide completed
    </label>

    Seřadit dle:
    <select v-model="selectedSortOption">
      <option value="default">--Vyberte--</option>
      <option value="name-asc">Názvu (ASC)</option>
      <option value="name-desc">Názvu (DESC)</option>
      <option value="creation-asc">Data přidání (ASC)</option>
      <option value="creation-desc">Data přidání (DESC)</option>
    </select>

    <ul>
      <li
          v-for="todo in todos"
          :key="todo.id"
          :class="{ 'todo--completed': todo.completed_at !== null }"
      >
        <label :for="`todo_${todo.id}`">
          <input
              type="checkbox"
              :checked="todo.completed_at !== null"
              :id="`todo_${todo.id}`"
              @change="toggleItem(todo.id)"
          >
          {{ todo.name }}

          ({{ todo.created_at }})
        </label>
      </li>
    </ul>
  </main>
</template>

<style scoped>
.todo--completed {
  text-decoration: line-through;
  color: gray;
}
</style>
