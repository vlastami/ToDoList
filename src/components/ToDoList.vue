<script setup>
import { parseISO, format } from 'date-fns';
import { defineProps, defineEmits } from 'vue';

defineProps({
  todos: {
    type: Array,
    required: false,
    default: () => []
  }
});

const emmit = defineEmits({
  toggle: null
});
</script>

<template>
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
            @change="emmit('toggle', todo.id)"
        >
        ({{ todo.priority }})
        {{ todo.name }}
        ({{ format(parseISO(todo.created_at), 'dd. MM. yyyy') }})
      </label>
    </li>
  </ul>
</template>

<style>
.todo--completed {
  text-decoration: line-through;
  color: gray;
}
</style>
