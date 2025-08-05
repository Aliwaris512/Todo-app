<template>
  <v-card class="todo-background" :class="{'theme--dark': isDarkMode, 'theme--light': !isDarkMode}">
    <div class="overlay" :class="{'dark-overlay': isDarkMode, 'light-overlay': !isDarkMode}"></div>
    <v-slide-x-transition>
      <div 
        :key="currentBackgroundImage"
        class="background-image"
        :style="{ backgroundImage: `url(${currentBackgroundImage})` }"
      ></div>
    </v-slide-x-transition>

    <v-toolbar :color="isDarkMode ? 'black' : 'white'">
      <img :src="logo" alt="Logo" height="100px" class="ml-2"/>
      <v-toolbar-title :class="{'white--text': isDarkMode}">TODO-LIST</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-menu offset-y>
        <template v-slot:activator="{ on, attrs }">
          <v-btn icon v-bind="attrs" v-on="on">
            <v-icon>{{ isDarkMode ? 'mdi-history' : 'mdi-history' }}</v-icon>
          </v-btn>
        </template>
        <v-list>
          <v-list-item v-for="(task, index) in recentTasks" :key="index">
            <v-list-item-content>
              <v-list-item-title>{{ task.text }}</v-list-item-title>
              <v-list-item-subtitle>{{ task.date }}</v-list-item-subtitle>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-menu>

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
                <v-btn color="success" @click="selectFile(i)">
                  {{ task.file ? 'Change PDF' : 'Add PDF' }}
                </v-btn>
                <span v-if="task.file" class="ml-2">
                  {{ task.file.name }}
                  <v-btn icon small @click="openPDF(task)">
                    <v-icon>mdi-open-in-new</v-icon>
                  </v-btn>
                </span>
              </v-list-item-subtitle>
              <template v-slot:append>
                <v-expand-x-transition>
                  <v-icon v-if="task.done" color="success">mdi-check</v-icon>
                </v-expand-x-transition>
                <v-btn
                  icon="mdi-pencil"
                  @click="editTask(task)"
                ></v-btn>
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
        <v-text-field
          v-model="email"
          label="Email Address"
          type="email"
          placeholder="Enter your email address"
        ></v-text-field>
        <v-btn
          v-if="email"
          color="success"
          @click="sendEmail"
        >
          Send Mail
        </v-btn>
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

    <input type="file" ref="fileInput" @change="attachFile" accept=".pdf" style="display:none;" />
  </v-card>
</template>
<style scoped>
.todo-background {
  position: relative;
  min-height: 100vh;
  overflow: hidden;
}

.background-image {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-size: cover;
  background-position: center;
  z-index: -1; 
  transition: opacity 0.1s ease-in-out;
}

.overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 1;
}

.light-overlay {
  background-color: rgba(255, 255, 255, 0);
}

.dark-overlay {
  background-color: rgba(0, 0, 0, 0.507);
}

.theme--dark {
  color: #fff7f7;
}

.theme--light {
  color: #000000;
}

.v-card {
  background-color: darkslateblue !important;
}

.v-container, .v-toolbar {
  position: relative;
  z-index: 1;
}
</style>
<script>
import jsPDF from 'jspdf';
import emailjs from '@emailjs/browser';
import Logo from "./assets/todo.jpg";
import TodoBackground1 from './assets/todo-background1.jpg';
import TodoBackground2 from './assets/todo-background2.jpg';
import TodoBackground3 from './assets/todo-background3.jpg';

emailjs.init('XevxtJzKnbspld8oC');

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
      backgrounds: [TodoBackground1, TodoBackground2, TodoBackground3],
      currentBackgroundIndex: 0,
      email: '', 
      recentTasks: [], 
    };
  },
  computed: {
    currentBackgroundImage() {
      return this.backgrounds[this.currentBackgroundIndex];
    },
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
    sendpdf() {
      const pdf = new jsPDF();
      pdf.text('hello', 10, 10);
      pdf.save('info.pdf');
    },
    cycleBackground() {
      this.currentBackgroundIndex = (this.currentBackgroundIndex + 1) % this.backgrounds.length;
    },
    saveTasks() {
      const tasksToSave = this.tasks.map(task => ({
        ...task,
        file: task.file ? { name: task.file.name, type: task.file.type } : null
      }));
      localStorage.setItem('tasks', JSON.stringify(tasksToSave));
    },
    loadTasks() {
      try {
        const savedTasks = localStorage.getItem('tasks');
        if (savedTasks) {
          const parsedTasks = JSON.parse(savedTasks);
          if (Array.isArray(parsedTasks)) {
            this.tasks = parsedTasks;
            this.recentTasks = parsedTasks.slice(-5).reverse();
          } else {
            console.error('Error: The saved tasks are not in the expected array format.');
            this.tasks = [];
            this.recentTasks = [];
          }
        }
      } catch (error) {
        console.error('Error loading tasks:', error);
        this.tasks = [];
        this.recentTasks = [];
      }
    },
    create() {
      if (this.newTask) {
        const newTask = {
          done: false,
          text: this.newTask,
          file: null,
          date: new Date().toLocaleString(),
        };
        this.tasks.push(newTask);
        this.updateRecentTasks(newTask); 
        this.newTask = null;
        this.saveTasks();
      }
    },
    updateRecentTasks(newTask) {
      this.recentTasks.unshift(newTask);
      if (this.recentTasks.length > 5) {
        this.recentTasks.pop();
      }
    },
    sendEmail() {
      if (!this.email) {
        alert('Please enter an email address.');
        return;
      }
      const tasksWithFiles = this.tasks.map(task => ({
        ...task,
        file: task.file ? { name: task.file.name, type: task.file.type, base64: task.file.base64 } : null
      }));
      const tasksText = tasksWithFiles.map(task => {
        const fileInfo = task.file ? ` (Attached: ${task.file.name})` : '';
        return `${task.text} - ${task.done ? 'Completed' : 'Pending'}${fileInfo}`;
      }).join('\n');

      const emailParams = {
        to_email: this.email,
        from_name: 'Ali To Do List App',
        subject: 'Tasks and PDFs',
        message: tasksText,
      };
      emailjs.init('OT-En7JP1B58_ATh2');
      emailjs.send('service_v86kosz', 'template_ll4de1p', emailParams)
        .then(response => {
          console.log('Email sent successfully:', response);
          this.dialogTitle = 'Success';
          this.dialogText = 'Tasks and PDFs have been sent successfully!';
          this.dialog = true;
          this.email = ''; 
          this.tasks = []; 
          this.saveTasks();
        })
        .catch(error => {
          console.error('Error sending email:', error);
          this.dialogTitle = 'Error';
          this.dialogText = 'There was an error sending the email.';
          this.dialog = true;
        });
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
        if (task.file && task.file.url) {
          URL.revokeObjectURL(task.file.url);
        }
        this.tasks.splice(index, 1);
        this.saveTasks();
      }
    },
    editTask(task) {
      const index = this.tasks.indexOf(task);
      if (index > -1) {
        const newText = prompt('Edit task:', task.text);
        if (newText !== null && newText.trim() !== '') {
          this.tasks[index].text = newText.trim();
          this.saveTasks();
        }
      }
    },
    selectFile(index) {
      this.selectedTaskIndex = index;
      this.$refs.fileInput.click();
    },
    attachFile(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          const base64 = e.target.result.split(',')[1]; 
          const url = URL.createObjectURL(file);
          this.tasks[this.selectedTaskIndex].file = {
            name: file.name,
            type: file.type,
            url: url,
            base64: base64 
          };
          this.saveTasks();
        };
        reader.readAsDataURL(file); 
      }
    },
    openPDF(task) {
      if (task.file && task.file.url) {
        window.open(task.file.url, '_blank');
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
    setInterval(this.cycleBackground, 5000); 
  },
  beforeUnmount() {
    this.tasks.forEach(task => {
      if (task.file && task.file.url) {
        URL.revokeObjectURL(task.file.url);
      }
    });
  },
};
</script>