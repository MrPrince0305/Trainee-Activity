<template>
  <q-page padding>
    <q-form @submit.prevent="void addTask()" class="q-gutter-sm flex">
      <q-input
        v-model="newTask.title"
        label="Task Name"
        filled
        dense
        class="col"
      />
      <q-btn
        label="Add Task"
        type="submit"
        color="primary"
        no-caps
      />
    </q-form>

    <div v-if="tasks.length" class="q-mt-lg">
      <q-list bordered separator>
        <q-item v-for="task in tasks" :key="task.id">
          <q-item-section>
            <div v-if="task.isEditing" class="flex items-center q-gutter-sm">
              <q-input
                v-model="task.title"
                dense
                autofocus
                @keyup.enter="void saveTask(task)"
                @blur="void cancelEdit(task)"
                class="col"
              />
              <q-btn
                label="Save"
                color="positive"
                dense
                flat
                no-caps
                @click="void saveTask(task)"
                @mousedown.prevent
              />
              <q-btn
                label="Cancel"
                color="negative"
                dense
                flat
                no-caps
                @click="void cancelEdit(task)"
                @mousedown.prevent
              />
            </div>
            <div v-else>
              <span>{{ task.title }}</span>
            </div>
          </q-item-section>

          <q-item-section side>
            <div class="flex items-center q-gutter-xs">
              <q-btn
                v-if="!task.isEditing"
                flat
                icon="edit"
                @click="void editTask(task)"
                color="grey"
                dense
              />
              <q-btn
                v-if="!task.isEditing"
                flat
                icon="delete"
                @click="void deleteTask(task.id)"
                color="negative"
                dense
              />
            </div>
          </q-item-section>
        </q-item>
      </q-list>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import axios from 'axios';

// Define the Task interface
interface Task {
  id: number;
  title: string;
  completed: boolean;
  isEditing?: boolean;
}

// Use the Task interface for types
const tasks = ref<Task[]>([]);
const newTask = ref<Task>({ title: '', completed: false, id: 0 });

// Reactive variable to store the original title of the task being edited
const originalEditingTitle = ref<string | null>(null);
// Reactive variable to store the ID of the task being edited (optional, but helps find it)
const editingTaskId = ref<number | null>(null);

// Base API URL
const API_URL = 'https://jsonplaceholder.typicode.com/todos';

// Get tasks (Read)
const getTasks = async () => {
  const response = await axios.get<Task[]>(API_URL + '?_limit=10');
  tasks.value = response.data.map(task => ({ ...task, isEditing: false }));
};

// Add task (Create)
const addTask = async () => {
  if (!newTask.value.title.trim()) return;
  const response = await axios.post<Task>(API_URL, {
    title: newTask.value.title,
    completed: false,
  });
  tasks.value.unshift({ ...response.data, isEditing: false });
  newTask.value.title = '';
};

// Function to enter edit mode
const editTask = (task: Task) => {
  // If another task is already being edited, cancel it first (good UX)
  if (editingTaskId.value !== null) {
    const currentlyEditingTask = tasks.value.find(t => t.id === editingTaskId.value);
    if (currentlyEditingTask) {
      cancelEdit(currentlyEditingTask);
    }
  }

  task.isEditing = true;
  // Store the original title and task ID
  originalEditingTitle.value = task.title;
  editingTaskId.value = task.id;
};

// Function to save the edited task
const saveTask = async (task: Task) => {
  if (!task.title.trim()) {
    // Handle empty title case
    cancelEdit(task); // Revert to original title if empty
    return;
  }

  // Call updateTask API function
  await updateTask(task);

  // Exit edit mode and clear original title state
  task.isEditing = false;
  originalEditingTitle.value = null;
  editingTaskId.value = null;
};

// Function to cancel editing
const cancelEdit = (task: Task) => {
  // Revert to original title
  if (originalEditingTitle.value !== null) {
    task.title = originalEditingTitle.value;
  }

  // Exit edit mode and clear original title state
  task.isEditing = false;
  originalEditingTitle.value = null;
  editingTaskId.value = null;
};

// Update task (Update) - This function remains largely the same, called by saveTask
const updateTask = async (task: Task) => {
  // Note: JSONPlaceholder doesn't persist changes for PUT, but we perform the action.
  await axios.put(`${API_URL}/${task.id}`, {
    title: task.title,
    completed: task.completed
  });
  // The UI is already updated via v-model in the template when editing
};

// Delete task (Delete)
const deleteTask = async (id: number) => {
  await axios.delete(`${API_URL}/${id}`);
  tasks.value = tasks.value.filter(task => task.id !== id);
};

// Fetch on component mount
onMounted(() => {
  void getTasks();
});
</script>

