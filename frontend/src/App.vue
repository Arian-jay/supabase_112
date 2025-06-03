<template>
  <v-app>
    <v-container class="ma-4" style="max-width: 600px;">
      <v-card class="pa-4">
        <v-card-title class="text-h5">To-Do List</v-card-title>
        <v-card-text>
          <!-- Add Todo -->
          <v-text-field
            v-model="newTask"
            label="Add a new task"
            variant="outlined"
            clearable
            @keyup.enter="addTodo"
            :disabled="loading"
            class="mb-4"
            prepend-inner-icon="mdi-plus"
          ></v-text-field>

          <!-- Todo List -->
          <v-list v-if="todos.length">
            <v-list-item
              v-for="todo in todos"
              :key="todo.id"
              class="mb-2"
            >
              <v-row align="center">
                <v-col cols="1">
                  <v-checkbox
                    v-model="todo.is_completed"
                    @change="toggleTodo(todo)"
                    :disabled="loading || editingTodoId === todo.id"
                  ></v-checkbox>
                </v-col>
                <v-col cols="7">
                  <v-text-field
                    v-if="editingTodoId === todo.id"
                    v-model="editingTask"
                    variant="outlined"
                    hide-details
                    @keyup.enter="saveEdit(todo)"
                    :disabled="loading"
                  ></v-text-field>
                  <span
                    v-else
                    :class="{ 'text-decoration-line-through': todo.is_completed }"
                    class="text-body-1"
                  >
                    {{ todo.task }}
                  </span>
                </v-col>
                <v-col cols="4" class="text-right">
                  <v-btn
                    v-if="editingTodoId === todo.id"
                    color="primary"
                    variant="text"
                    @click="saveEdit(todo)"
                    :disabled="loading || !editingTask.trim()"
                  >
                    Save
                  </v-btn>
                  <v-btn
                    v-else
                    color="primary"
                    variant="text"
                    @click="startEdit(todo)"
                    :disabled="loading"
                  >
                    Edit
                  </v-btn>
                  <v-btn
                    color="error"
                    variant="text"
                    @click="confirmDelete(todo.id)"
                    :disabled="loading"
                  >
                    Delete
                  </v-btn>
                </v-col>
              </v-row>
            </v-list-item>
          </v-list>
          <v-alert
            v-if="!todos.length && !error"
            type="info"
            text="No todos yet. Add one above!"
            class="mt-4"
          ></v-alert>

          <!-- Clear All Button -->
          <v-btn
            v-if="todos.length"
            color="warning"
            variant="outlined"
            @click="confirmClearAll"
            :disabled="loading"
            class="mt-4"
          >
            Clear All Todos
          </v-btn>

          <!-- Error Message -->
          <v-alert
            v-if="error"
            type="error"
            :text="error"
            class="mt-4"
          ></v-alert>
        </v-card-text>
      </v-card>

      <!-- Delete Confirmation Dialog -->
      <v-dialog v-model="showDeleteDialog" max-width="400">
        <v-card>
          <v-card-title>Confirm Delete</v-card-title>
          <v-card-text>Are you sure you want to delete this todo?</v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn text @click="showDeleteDialog = false">Cancel</v-btn>
            <v-btn color="error" @click="deleteTodo">Delete</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- Clear All Confirmation Dialog -->
      <v-dialog v-model="showClearAllDialog" max-width="400">
        <v-card>
          <v-card-title>Confirm Clear All</v-card-title>
          <v-card-text>Are you sure you want to delete all todos?</v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn text @click="showClearAllDialog = false">Cancel</v-btn>
            <v-btn color="error" @click="clearAllTodos">Clear All</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-container>
  </v-app>
</template>

<script>
export default {
  data() {
    return {
      todos: [],
      newTask: '',
      error: '',
      loading: false,
      editingTodoId: null,
      editingTask: '',
      showDeleteDialog: false,
      deleteTodoId: null,
      showClearAllDialog: false,
    };
  },
  mounted() {
    this.fetchTodos();
  },
  methods: {
    async fetchTodos() {
      this.loading = true;
      try {
        const response = await fetch('http://localhost:3000/todos');
        if (!response.ok) throw new Error('Failed to fetch todos');
        this.todos = await response.json();
        this.error = '';
      } catch (err) {
        this.error = 'Could not connect to backend. Please ensure it is running.';
        console.error(err);
      } finally {
        this.loading = false;
      }
    },
    async addTodo() {
      if (!this.newTask.trim() || this.loading) return;
      this.loading = true;
      try {
        const response = await fetch('http://localhost:3000/todos', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ task: this.newTask })
        });
        if (!response.ok) throw new Error('Failed to add todo');
        const newTodo = await response.json();
        this.todos.unshift(newTodo);
        this.newTask = '';
        this.error = '';
      } catch (err) {
        this.error = 'Could not connect to backend. Please ensure it is running.';
        console.error(err);
      } finally {
        this.loading = false;
      }
    },
    startEdit(todo) {
      this.editingTodoId = todo.id;
      this.editingTask = todo.task;
    },
    async saveEdit(todo) {
      if (!this.editingTask.trim() || this.loading) return;
      this.loading = true;
      try {
        const response = await fetch(`http://localhost:3000/todos/${todo.id}`, {
          method: 'PUT',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ task: this.editingTask, is_completed: todo.is_completed })
        });
        if (!response.ok) throw new Error('Failed to update todo');
        const updatedTodo = await response.json();
        this.todos = this.todos.map(t => (t.id === todo.id ? updatedTodo : t));
        this.editingTodoId = null;
        this.editingTask = '';
        this.error = '';
      } catch (err) {
        this.error = 'Could not connect to backend. Please ensure it is running.';
        console.error(err);
      } finally {
        this.loading = false;
      }
    },
    async toggleTodo(todo) {
      if (this.loading) return;
      this.loading = true;
      try {
        const response = await fetch(`http://localhost:3000/todos/${todo.id}`, {
          method: 'PUT',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ task: todo.task, is_completed: todo.is_completed })
        });
        if (!response.ok) throw new Error('Failed to update todo');
        const updatedTodo = await response.json();
        this.todos = this.todos.map(t => (t.id === todo.id ? updatedTodo : t));
        this.error = '';
      } catch (err) {
        this.error = 'Could not connect to backend. Please ensure it is running.';
        console.error(err);
        // Revert checkbox state on error
        todo.is_completed = !todo.is_completed;
      } finally {
        this.loading = false;
      }
    },
    confirmDelete(id) {
      this.deleteTodoId = id;
      this.showDeleteDialog = true;
    },
    async deleteTodo() {
      if (this.loading) return;
      this.loading = true;
      try {
        const response = await fetch(`http://localhost:3000/todos/${this.deleteTodoId}`, {
          method: 'DELETE'
        });
        if (!response.ok) throw new Error('Failed to delete todo');
        this.todos = this.todos.filter(todo => todo.id !== this.deleteTodoId);
        this.error = '';
      } catch (err) {
        this.error = 'Could not connect to backend. Please ensure it is running.';
        console.error(err);
      } finally {
        this.showDeleteDialog = false;
        this.deleteTodoId = null;
        this.loading = false;
      }
    },
    confirmClearAll() {
      this.showClearAllDialog = true;
    },
    async clearAllTodos() {
      if (this.loading) return;
      this.loading = true;
      try {
        for (const todo of this.todos) {
          const response = await fetch(`http://localhost:3000/todos/${todo.id}`, {
            method: 'DELETE'
          });
          if (!response.ok) throw new Error('Failed to delete todo');
        }
        this.todos = [];
        this.error = '';
      } catch (err) {
        this.error = 'Could not connect to backend. Please ensure it is running.';
        console.error(err);
      } finally {
        this.showClearAllDialog = false;
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped>
.text-decoration-line-through {
  text-decoration: line-through;
}
</style>