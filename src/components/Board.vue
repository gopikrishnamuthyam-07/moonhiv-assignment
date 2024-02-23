<template>
  <div>
    <div class="boards">
      <div v-for="(project, index) in projects" :key="index" class="board">
        <div class="text-center uppercase">
          <b>{{ project.name }}</b>
        </div>
        <div class="text-end">
          <button
            @click="openModal(index)"
            class="bg-green-500 px-1 hover:bg-green-700 text-white font-bold rounded"
          >
            +
          </button>
        </div>
        <div class="tasks">
          <body class="antialiased font-sans bg-gray-200">
            <div class="container mx-auto px-4 sm:px-8">
              <div class="py-8">
                <div class="-mx-4 sm:-mx-8 px-4 sm:px-8 py-4 overflow-x-auto">
                  <table class="min-w-full leading-normal border-collapse">
                    <thead>
                      <tr>
                        <th></th>
                        <th
                          v-for="(category, catIndex) in project.categories"
                          :key="catIndex"
                          class="px-5 py-3 border-b-2 border-gray-200 bg-slate-400 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider"
                        >
                          {{ category.name }}
                        </th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr
                        v-for="(task, taskIndex) in getMaxCategoryLength(
                          project.categories
                        )"
                        :key="taskIndex"
                      >
                        <td class="px-5 py-5 border-b border-gray-200 text-sm">
                          <div class="flex items-center">
                            <div class="ml-3">
                              <p class="text-gray-900 whitespace-no-wrap">
                                {{ task ? task.name : "" }}
                              </p>
                            </div>
                          </div>
                        </td>
                        <template
                          v-for="(category, catIndex) in project.categories"
                          :key="catIndex"
                        >
                          <td
                            class="px-5 py-5 border-b border-gray-200 text-sm hover:cursor-move"
                            :class="{
                              [category.color]:
                                task && taskIndex < category.tasks.length,
                            }"
                          >
                            <draggable
                              v-if="category.tasks"
                              v-model="category.tasks"
                              group="tasks"
                              @end="onTaskDragEnd(index, catIndex)"
                            >
                              <template #item="{ element }">
                                <div class="text-center task-item">
                                  <p
                                    class="text-gray-900 whitespace-no-wrap p-2"
                                  >
                                    {{ element.name }}
                                  </p>
                                </div>
                              </template>
                            </draggable>
                          </td>
                        </template>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </div>
            </div>
          </body>
        </div>
      </div>
    </div>
    <div v-if="showModal" class="modal-overlay">
      <div
        class="modal fixed w-full h-full top-0 left-0 flex items-center justify-center"
      >
        <div class="modal-container bg-white w-1/3 rounded-lg p-4">
          <button class="close-button" @click="closeModal">
            <svg
              class="fill-current text-black"
              xmlns="http://www.w3.org/2000/svg"
              width="18"
              height="18"
              viewBox="0 0 18 18"
            >
              <path
                d="M14.848.768c-.576-.576-1.536-.576-2.112 0L9 6.888 5.264 3.152c-.576-.576-1.536-.576-2.112 0s-.576 1.536 0 2.112L6.888 9l-3.736 3.736c-.576.576-.576 1.536 0 2.112.288.288.672.432 1.056.432s.768-.144 1.056-.432L9 11.112l3.736 3.736c.288.288.672.432 1.056.432s.768-.144 1.056-.432c.576-.576.576-1.536 0-2.112L11.112 9l3.736-3.736c.576-.576.576-1.536 0-2.112z"
              />
            </svg>
          </button>
          <input
            v-model="taskName"
            @keyup.enter="addTask"
            class="rounded-md p-2 border border-gray-300 mb-2 w-full"
            placeholder="Enter task name"
          />
          <div class="text-center">
            <button
              @click="addTask"
              class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
            >
              Add Task
            </button>
          </div>
        </div>
      </div>
    </div>
    <div class="text-center">
      <button
        class="rounded-lg p-1 text-white bg-blue-500 hover:bg-blue-800"
        @click="createBoard"
      >
        Create Board
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import Draggable from "vuedraggable";

const projects = ref([
  {
    name: "Project 1",
    categories: [
      { name: "To Do", color: "bg-red-500", tasks: [] },
      { name: "In Progress", color: "bg-slate-500", tasks: [] },
      { name: "Done", color: "bg-green-500", tasks: [] },
    ],
  },
]);

const showModal = ref(false);
const taskName = ref("");
let selectedProjectIndex = null;

const createBoard = () => {
  const newProject = {
    name: `Project ${projects.value.length + 1}`,
    categories: [
      { name: "To Do", color: "bg-red-500", tasks: [] },
      { name: "In Progress", color: "bg-blue-500", tasks: [] },
      { name: "Done", color: "bg-green-500", tasks: [] },
    ],
  };
  projects.value.push(newProject);
};

const openModal = (projectIndex) => {
  showModal.value = true;
  selectedProjectIndex = projectIndex;
};

const closeModal = () => {
  showModal.value = false;
};

const addTask = () => {
  const name = taskName.value.trim();
  if (name !== "") {
    projects.value[selectedProjectIndex].categories[0].tasks.push({ name });
    taskName.value = "";
    showModal.value = false;
  }
};

const onTaskDragEnd = (projectIndex, categoryIndex) => (event) => {
  const { newIndex, oldIndex } = event;
  const movedTask = projects.value[projectIndex].categories[
    categoryIndex
  ].tasks.splice(oldIndex, 1)[0];
  projects.value[projectIndex].categories[categoryIndex].tasks.splice(
    newIndex,
    0,
    movedTask
  );
};

const getMaxCategoryLength = (categories) => {
  let maxLength = 0;
  categories.forEach((category) => {
    if (category.tasks.length > maxLength) {
      maxLength = category.tasks.length;
    }
  });
  return new Array(maxLength).fill(null);
};
</script>

<style scoped>
.board {
  flex: 0 0 30%;
  margin: 10px;
  padding: 15px;
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  border-radius: 5px;
}

.boards {
  display: flex;
  flex-wrap: wrap;
}

.board {
  flex: 1;
  margin: 10px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.categories {
  margin-top: 10px;
}

.category {
  margin-bottom: 5px;
  padding: 5px;
  border-radius: 5px;
}

.tasks {
  margin-top: 10px;
}

.task {
  margin-bottom: 5px;
  padding: 5px;
  border-radius: 5px;
}

.text-center {
  text-align: center;
}

.task-item {
  background-color: #cccccc;
  /* Grey background color for task items */
}
</style>
