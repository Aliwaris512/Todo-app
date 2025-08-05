<template>
  <v-card :class="{'theme--dark': isDarkMode, 'theme--light': !isDarkMode}">
    <v-toolbar :color="isDarkMode ? 'black' : 'black'">
      <img :src="logo" alt="Logo" height="100px" class="ml-2"/>
      <v-toolbar-title>TODO-LIST</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn @click="toggleDarkMode" icon>
        <v-icon>{{ isDarkMode ? 'mdi-lightbulb-on' : 'mdi-lightbulb-off' }}</v-icon>
      </v-btn>
    </v-toolbar>
    <v-container style="max-width: 500px"></v-container>
    <v-container>
      <v-text-field
        v-model="newTask"
        label="What are you working on?"
        variant="solo"
        @keydown.enter="create"
      >
        <template v-slot:append-inner>
          <v-fade-transition>
            <v-btn
              v-show="newTask"
              icon="mdi-plus-circle"
              variant="text"
              @click="create"
            ></v-btn>
          </v-fade-transition>
        </template>
      </v-text-field>

      <h2 class="text-h4 text-success ps-4">
        Tasks:&nbsp;
        <v-fade-transition leave-absolute>
          <span :key="`tasks-${tasks.length}`">
            {{ tasks.length }}
          </span>
        </v-fade-transition>
      </h2>

      <v-divider class="mt-4"></v-divider>

      <v-row align="center" class="my-1">
        <strong class="mx-4 text-info-darken-2">
          Remaining: {{ remainingTasks }}
        </strong>
        <v-divider vertical></v-divider>
        <strong class="mx-4 text-success-darken-2">
          Completed: {{ completedTasks }}
        </strong>
        <v-spacer></v-spacer>
        <v-progress-circular
          :value="progress"
          :size="40"
          :width="5"
          color="primary"
          class="me-2"
        ></v-progress-circular>
      </v-row>

      <v-divider class="mb-4"></v-divider>

      <v-card v-if="tasks.length > 0">
        <v-slide-y-transition class="py-0" tag="v-list" group>
          <template v-for="(task, i) in tasks" :key="`${i}-${task.text}`">
            <v-divider v-if="i !== 0" :key="`${i}-divider`"></v-divider>
            <v-list-item>
              <template v-slot:prepend>
                <v-checkbox-btn v-model="task.done" color="black"></v-checkbox-btn>
              </template>
              <v-list-item-title>
                <span :class="task.done ? 'text-black' : 'text-primary'">{{ task.text }}</span>
              </v-list-item-title>
              <v-list-item-subtitle>
                <v-row justify="center" class="mt-4"></v-row>
                <v-btn  colour="sucess" @click="selectFile(i)">
                   PDF
                </v-btn>
                <span v-if="task.file">{{ task.file.name }}</span>
              </v-list-item-subtitle>
              <template v-slot:append>
                <v-expand-x-transition>
                  <v-icon v-if="task.done" color="success">mdi-check</v-icon>
                </v-expand-x-transition>
                <v-btn
                  icon="mdi-delete"
                  @click="deleteTask(task)"
                ></v-btn>
              </template>
            </v-list-item>
          </template>
        </v-slide-y-transition>
      </v-card>

      <v-row justify="center" class="mt-4">
        <v-btn
          v-if="allTasksCompleted"
          color="success"
          @click="handleSubmit"
        >
          Submit
        </v-btn>
      </v-row>
    </v-container>

    <v-dialog v-model="dialog" max-width="290">
      <v-card>
        <v-card-title class="headline">{{ dialogTitle }}</v-card-title>
        <v-card-text>
          {{ dialogText }}
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" text @click="dialog = false">OK</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Hidden file input for selecting PDF files -->
    <input type="file" ref="fileInput" @change="attachFile" accept=".pdf" style="display:none;" />
  </v-card>
</template>
<style scoped>
.theme--dark {
  background-color: #121212; 
  color: #ffffff;
}

.theme--light {
  background-color: #ffffff; 
  color: #000000; 
}
</style>
<script>
import Logo from "./assets/todo.jpg"
export default {
  data() {
    return {
      tasks: [],
      newTask: null,
      dialog: false,
      dialogTitle: '',
      dialogText: '',
      logo: Logo,
      isDarkMode: false,
      selectedTaskIndex: null, 
    };
  },
  computed: {
    completedTasks() {
      return this.tasks.filter(task => task.done).length;
    },
    progress() {
      return this.tasks.length > 0 ? (this.completedTasks / this.tasks.length) * 100 : 0;
    },
    remainingTasks() {
      return this.tasks.length - this.completedTasks;
    },
    allTasksCompleted() {
      return this.tasks.length > 0 && this.tasks.every(task => task.done);
    },
  },
  methods: {
    saveTasks() {
      localStorage.setItem('tasks', JSON.stringify(this.tasks));
    },
    loadTasks() {
      const savedTasks = localStorage.getItem('tasks');
      if (savedTasks) {
        this.tasks = JSON.parse(savedTasks);
      }
    },
    create() {
      if (this.newTask) {
        this.tasks.push({
          done: false,
          text: this.newTask,
          file: null, // Add file property to each task
        });
        this.newTask = null;
        this.saveTasks(); 
      }
    },
    toggleDarkMode() {
      this.isDarkMode = !this.isDarkMode;
      this.$vuetify.theme.dark = this.isDarkMode; 
    },
    handleSubmit() {
      this.dialogTitle = 'Submit Successful';
      this.dialogText = 'Your tasks have been submitted successfully!';
      this.dialog = true;
      this.saveTasksToFile();
      localStorage.removeItem('tasks');  
      this.tasks = [];
    },
    deleteTask(task) {
      const index = this.tasks.indexOf(task);
      if (index > -1) {
        this.tasks.splice(index, 1);
        this.saveTasks();  
      }
    },
    selectFile(index) {
      this.selectedTaskIndex = index;
      this.$refs.fileInput.click(); // Trigger file input click
    },
    attachFile(event) {
      const file = event.target.files[0];
      if (file) {
        this.tasks[this.selectedTaskIndex].file = file;
        this.saveTasks(); 
      }
    },
    saveTasksToFile() {
      const taskText = this.tasks.map(task => {
        const fileInfo = task.file ? ` (Attached: ${task.file.name})` : '';
        return `${task.text} - ${task.done ? 'Completed' : 'Pending'}${fileInfo}`;
      }).join('\n');
      const blob = new Blob([taskText], { type: 'text/plain' });
      const link = document.createElement('a');
      link.href = URL.createObjectURL(blob);
      link.download = 'tasks.txt'; 
      link.click();
      URL.revokeObjectURL(link.href);
    },
  },
  mounted() {
    this.loadTasks();  
  },
};
</script>
