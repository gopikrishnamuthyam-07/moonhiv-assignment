<template>
    <div>
        <div class="boards">
            <!-- Dashboard view -->
            <div v-for="(project, index) in projects" :key="index" class="board" :class="{ 'archived': project.archived }">
                <div class="text-center uppercase">
                    <b>{{ project.name }}</b>
                    <!-- Button to edit project name -->
                    <button @click="editProjectName(index)" class="ml-2">
                        edit 
                    </button>
                    <!-- Button to archive project -->
                    <button @click="toggleArchiveProject(index)" class="ml-4 pt-4">
                        <img v-if="project.archived" class="h-auto float-right w-auto max-h-4  max-w-full"
                            src="C:\Users\GopiKrishnaMuthyam\VUE PROJECTS\DEMO-VUE\src\assets\close eye.png" alt="Archived">
                        <img v-else src="C:\Users\GopiKrishnaMuthyam\VUE PROJECTS\DEMO-VUE\src\assets\open eye 2.png"
                            alt="Not Archived" class="h-auto float-right w-auto max-h-4  max-w-full">
                    </button>

                </div>
                <div class="text-end">
                    <button @click="openModal(index)"
                        class="bg-green-500 px-1 hover:bg-green-700 text-white font-bold rounded">
                        +
                    </button>
                </div>
                <!-- Lists view -->
                <div class="tasks">
                    <div v-for="(category, catIndex) in project.categories" :key="catIndex" class="category flex flex-col">
                        <span class="flex flex-col">
                            <button @click="editListName(index, catIndex)">
                                <img src="../assets/download.png" class="h-auto float-right w-auto max-h-4  max-w-full">
                            </button>
                            <input v-model="category.name"
                                class="list-name float-start text-center uppercase font-semibold bg-slate-400 hover:bg-slate-800 hover:text-white"
                                @blur="updateListName(index, catIndex)" />
                            <!-- Button to edit list name -->
                        </span>
                        <draggable v-model="category.tasks" group="tasks" @end="onTaskDragEnd(index, catIndex)">
                            <template #item="{ element }">
                                <li class="task flex justify-between">
                                    <!-- Input field to edit task name -->
                                    <input v-model="element.name" class="task-name text-start text-blue-700"
                                        @blur="updateTaskName(index, catIndex, element)" />
                                    <button @click="deleteTask(index, catIndex, element)" class="text-end">
                                        <img src="../assets/delete-icon-1864x2048-bp2i0gor.png"
                                            class="h-auto w-auto max-h-4  max-w-full">
                                    </button>
                                </li>
                            </template>
                        </draggable>
                    </div>
                </div>
            </div>
        </div>
        <div v-if="showModal" class="modal-overlay">
            <div class="modal fixed w-full h-full top-0 left-0 flex items-center justify-center">
                <div class="modal-container bg-white w-1/3 rounded-lg p-4">
                    <div class="text-end">
                        <button class="bg-red-400 rounded-lg hover:bg-red-800  px-2 py-1 mb-1 text-white"
                            @click="closeModal">
                            <img src="../assets/images.png" class="h-auto w-auto max-h-4  max-w-full">
                        </button>
                    </div>
                    <input v-model="taskName" @keyup.enter="addTask"
                        class="rounded-md p-2 border border-gray-300 mb-2 w-full" placeholder="Enter task name" />
                    <div class="text-center">
                        <button @click="addTask"
                            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-1 px-2  rounded">
                            Add Task
                        </button>
                    </div>
                </div>
            </div>
        </div>
        <!-- Button to create new board -->
        <div class="text-center mt-4">
            <button class="rounded-lg p-1 text-white bg-blue-500 hover:bg-blue-800" @click="createBoard">
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
        archived: false, // New property to track archived state
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
        archived: false, // New property to track archived state
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

const updateListName = (projectIndex, listIndex) => {
    // Update the name of the list in the project
    const newName = projects.value[projectIndex].categories[listIndex].name;
    console.log("Updated list name:", newName);
};

const editListName = (projectIndex, listIndex) => {
    // Retrieve the list object
    const list = projects.value[projectIndex].categories[listIndex];

    // Open an input field or modal for editing the list name
    const newListName = prompt("Enter new list name:", list.name);

    // Update the list name if a new name is provided
    if (newListName !== null) {
        list.name = newListName.trim();
        console.log("List name updated:", newListName);
    }
};

const editProjectName = (projectIndex) => {
    // Retrieve the project object
    const project = projects.value[projectIndex];

    // Open an input field or modal for editing the project name
    const newProjectName = prompt("Enter new project name:", project.name);

    // Update the project name if a new name is provided
    if (newProjectName !== null) {
        project.name = newProjectName.trim();
        console.log("Project name updated:", newProjectName);
    }
};

const updateTaskName = (projectIndex, listIndex, taskIndex) => {
    // Retrieve the task object
    const task = projects.value[projectIndex].categories[listIndex].tasks[taskIndex];

    // Open an input field or modal for editing the task name
    const newTaskName = prompt("Enter new task name:", task.name);

    // Update the task name if a new name is provided
    if (newTaskName !== null) {
        task.name = newTaskName.trim();
        console.log("Task name updated:", newTaskName);
    }
};

const deleteTask = (projectIndex, listIndex, task) => {
    // Delete the task from the project
    const index = projects.value[projectIndex].categories[listIndex].tasks.indexOf(task);
    if (index !== -1) {
        projects.value[projectIndex].categories[listIndex].tasks.splice(index, 1);
    }
};

const toggleArchiveProject = (projectIndex) => {
    projects.value[projectIndex].archived = !projects.value[projectIndex].archived;
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

.archived {
    opacity:0.2;
    /* You can adjust the style for archived projects */
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
}</style>
