<template>
  <div v-show="showAddTask"><AddTask @add-task="addTask" /></div>

  <Tasks
    @toggle-reminder="toggleReminder"
    @delete-task="deleteTask"
    :tasks="tasks"
  />
</template>
<script>
import Tasks from "../components/Tasks.vue";
import AddTask from "../components/AddTask.vue";

export default {
  name: "Home",
  components: {
    Tasks,
    AddTask,
  },
  props: {
    showAddTask: Boolean,
  },
  data() {
    return {
      tasks: [],
    };
  },

  methods: {
    async addTask(newTask) {
      const res = await fetch("http://localhost:5002/tasks", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(newTask),
      });

      const data = await res.json();

      this.tasks = [...this.tasks, data];
    },
    async deleteTask(id) {
      if (confirm(`are you sure you want to delete?`)) {
        const res = await fetch(`http://localhost:5002/tasks/${id}`, {
          method: "DELETE",
        });

        res.status === 200
          ? (this.tasks = this.tasks.filter((task) => task.id !== id))
          : alert("Error deleting");
      }
    },

    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id);
      const updTask = { ...taskToToggle, reminder: !taskToToggle.reminder };
      const res = await fetch(`http://localhost:5002/tasks/${id}`, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(updTask),
      });

      const data = await res.json();

      this.tasks = this.tasks.map((task) =>
        task.id === id ? { ...task, reminder: data.reminder } : task
      );
    },

    async fetchTasks() {
      const res = await fetch("http://localhost:5002/tasks");
      const data = await res.json();
      return data;
    },
    async fetchTask(id) {
      const res = await fetch(`http://localhost:5002/tasks/${id}`);
      const data = await res.json();
      return data;
    },
  },
  async created() {
    this.tasks = await this.fetchTasks();
  },
};
</script>