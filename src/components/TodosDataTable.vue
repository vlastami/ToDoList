<script setup>
import { ref, onMounted, watch, defineProps, defineEmits } from 'vue';
import { format } from 'date-fns';

import TodoList from './TodoList.vue';

const props = defineProps({
  priority: {
    type: Number,
    required: false,
    default: 0
  },
  changes: {
    type: Number,
    required: false,
    default: -1
  }
});

const emmit = defineEmits({
  todosChanged: null
});

const todos = ref([]);
const sort = ref('name-asc');
const hideCompleted = ref(true);

const jsonHeaders = () => ({
  'Content-Type': 'application/json',
});

async function loadTodos() {
  let url = 'http://localhost:8000/api/todos';

  const params = [];

  if (sort.value !== 'default') {
    const [orderBy, direction] = sort.value.split('-');

    params.push(`order_by=${orderBy}`)
    params.push(`direction=${direction}`)
  }

  if (props.priority > 0) {
    params.push(`priority=${props.priority}`)
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

async function toggleItem(todoId) {
  const todo = todos.value.find((todo) => todo.id === todoId);

  await fetch(`http://localhost:8000/api/todos/${todoId}`, {
    method: 'PUT',
    headers: jsonHeaders(),
    body: JSON.stringify({
      'completed_at': todo.completed_at === null
          ? format(new Date(), 'yyyy-MM-dd HH:mm:ss')
          : null
    })
  });

  emmit('todosChanged')

  if (props.changes < 0) {
    await loadTodos();
  }
}

onMounted(loadTodos);

watch(sort, loadTodos);
watch(hideCompleted, loadTodos);
watch(() => props.changes, loadTodos);
</script>

<template>
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

  <TodoList
      :todos="todos"
      @toggle="toggleItem"
  />
</template>

<style>

</style>
