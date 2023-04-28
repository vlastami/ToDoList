<script>
const API_BASE_URL = "http://192.168.59.251:8000/api/todos";

export default {
  data() {
    return {
      newTodo: "",
      todos: [],
    };
  },
  async created() {
    try {
      const response = await fetch(API_BASE_URL);
      const data = await response.json();
      this.todos = data.todos;
    } catch (error) {
      console.error("Error fetching todos:", error);
    }
  },
  methods: {
    async addTodo() {
      try {
        const response = await fetch(API_BASE_URL, {
          method: "POST",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify({
            name: this.newTodo,
            priority: 1,
            order: this.todos.length + 1,
          }),
        });
        const data = await response.json();
        this.todos.push(data);
        this.newTodo = "";
      } catch (error) {
        console.error("Error adding todo:", error);
      }
    },
// udělat mazání
    // udělat updatedAt
    // připravil endpoint (query parameter) který schová udělané (defaultně)
    // chceme button show completed
    formatDate(dateString) {
      const date = new Date(dateString);
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, "0");
      const day = String(date.getDate()).padStart(2, "0");
    },

    async toggleComplete(todo) {
      try {
        const requestData = {
          completed_at: todo.completed_at ? null : new Date().toISOString(),
        };
        console.log("Request data:", requestData);

        const response = await fetch(`${API_BASE_URL}/${todo.id}`, {
          method: "PATCH",
          headers: {
            "Content-Type": "application/json",
          },
          body: JSON.stringify(requestData),
        });

        if (!response.ok) {
          console.log("Response status:", response.status, response.statusText);
          console.log("Response text:", await response.text());
          throw new Error("Error toggling todo completion: " + response.statusText);
        }

        const data = await response.json();
        todo.completed_at = data.completed_at;
      } catch (error) {
        console.error("Error toggling todo completion:", error);
      }
    },


  },
};
</script>

<template>
  <div id="app">
    <h1>ToDoList</h1>
    <form @submit.prevent="addTodo">
      <input type="text" v-model="newTodo" placeholder="Write a to do item" required />
      <button type="submit">Add</button>
    </form>
    <ul>
      <li v-for="todo in todos" :key="todo.id">
        <input type="checkbox" :checked="todo.completed_at" @change="toggleComplete(todo)" />
        <span :class="{ completed: todo.completed_at }">{{ todo.name }}</span>
        <span v-if="todo.completed_at"> (Completed on {{ formatDate(todo.completed_at) }})</span>
      </li>
    </ul>
  </div>
</template>


<style>
.completed {
  text-decoration: line-through;
  color: gray;
}
</style>
