<template>
    <div class="list">
      <div class="list-header">
        <h3>{{ list.name }}</h3>
        <button @click="editList">Edit</button>
        <button @click="archiveList">{{ list.archived ? 'Restore' : 'Archive' }}</button>
      </div>
      <div class="tasks">
        <div v-for="(task, index) in list.tasks" :key="index" class="task">
          {{ task.name }}
        </div>
        <button @click="addTask">Add Task</button>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    props: ['list'],
    methods: {
      addTask() {
        const taskName = prompt('Enter task name:');
        if (taskName) {
          this.list.tasks.push({ name: taskName });
        }
      },
      editList() {
        const newName = prompt('Enter new list name:', this.list.name);
        if (newName) {
          this.list.name = newName;
        }
      },
      archiveList() {
        this.list.archived = !this.list.archived;
      }
    }
  };
  </script>
  
  <style scoped>
  .list {
    border: 1px solid #ddd;
    border-radius: 5px;
    padding: 10px;
    margin-right: 20px;
  }
  
  .list-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 10px;
  }
  
  .tasks {
    margin-top: 10px;
  }
  
  .task {
    padding: 5px;
    margin-bottom: 5px;
    background-color: #f9f9f9;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
  </style>
  