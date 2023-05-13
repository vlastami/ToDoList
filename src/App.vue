<script setup>
import { ref, onMounted, watch } from 'vue';
import { format, parseISO } from 'date-fns';

const todos = ref([]);
const newTodo = ref('');
const sort = ref('name-desc');
const hideCompleted = ref(true);

function jsonHeaders() {
  return {
    'Content-Type': 'application/json',
  };
}

async function loadTodos() {
  let url = 'http://192.168.59.251:8000/api/todos';

  const params = [];

  if (sort.value !== 'default') {
    const [orderBy, direction] = sort.value.split('-');

    params.push(`order_by=${orderBy}`)
    params.push(`direction=${direction}`)
  }

  // ...

  params.push(`show_completed=${hideCompleted.value ? 0 : 1}`)

  if (params.length > 0) {
    url += '?' + params.join('&')
  }

  const response = await fetch(url);
  const body = await response.json();

  todos.value = body.todos;
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

watch(sort, () => {
  loadTodos();
})

watch(hideCompleted, () => {
  loadTodos();
})
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
          v-model="hideCompleted"
          type="checkbox"
          id="hide-completed"
      >

      Hide completed
    </label>

    Seřadit dle:
    <select v-model="sort">
      <option value="default">--Vyberte--</option>
      <option value="name-asc">Názvu (ASC)</option>
      <option value="name-desc">Názvu (DESC)</option>
      <option value="created_at-asc">Data přidání (ASC)</option>
      <option value="created_at-desc">Data přidání (DESC)</option>
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

          ({{ format(parseISO(todo.created_at), 'dd. MM. yyyy') }})
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