<script setup>
import HelloWorld from './components/HelloWorld.vue'
import { ref, computed } from 'vue';

const newTask = ref('');
const newTaskTime = ref('');
const newTaskTag = ref('');
const tasks = ref([]);
const newTaskPoints = ref(2);

const addTask = () => {
  if (newTask.value.trim()) {
    tasks.value.push({
      text: newTask.value.trim(),
      time: newTaskTime.value,
      tag: newTaskTag.value,
      points: parseInt(newTaskPoints.value) || 2,
      completed: false
    });
    newTask.value = '';
    newTaskTime.value = '';
    newTaskTag.value = '';
    newTaskPoints.value = 2;
  }
};

const deleteTask = (index) => {
  tasks.value.splice(index, 1);
};

const completeTask = (index) => {
  tasks.value[index].completed = !tasks.value[index].completed;
};
const totalPoints = computed(() => {
  return tasks.value.filter(task => task.completed).reduce((sum, task) => sum + task.points, 0);
});
</script>

<template>
  <div>
    <h1>任务清单</h1>
    <input v-model="newTask" placeholder="输入新任务" @keyup.enter="addTask" />
    <input v-model="newTaskTime" type="datetime-local" placeholder="设置任务时间" @keyup.enter="addTask" />
    <input v-model="newTaskTag" placeholder="添加标签" @keyup.enter="addTask" />
    <input v-model.number="newTaskPoints" type="number" placeholder="积分（默认2）" @keyup.enter="addTask" />
    <button @click="addTask">添加任务</button>
    <p>已完成任务积分总和: {{ totalPoints }}</p>
    <ul>
      <li v-for="(task, index) in tasks" :key="index" :style="{ textDecoration: task.completed ? 'line-through' : 'none' }">
        {{ task.text }}
        <span>时间: {{ task.time }}</span>
        <span>标签: {{ task.tag }}</span>
        <span>积分: {{ task.points }}</span>
        <button @click="deleteTask(index)">删除</button>
        <button @click="completeTask(index)">{{ task.completed ? '取消完成' : '标记完成' }}</button>
      </li>
    </ul>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}

/* 新增样式 */
body {
  font-family: Arial, sans-serif;
  background-color: #f4f4f9;
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  margin: 0;
}

div {
  background-color: white;
  padding: 2rem;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 600px;
}

input, button {
  margin: 0.5rem;
  padding: 0.5rem;
  border: 1px solid #ddd;
  border-radius: 4px;
}

button {
  background-color: #646cff;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #535bf2;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: flex;
  align-items: center;
  padding: 0.5rem;
  margin: 0.5rem 0;
  border-radius: 4px;
  transition: background-color 0.3s;
  color: #000; 
}

li:hover {
  background-color: #f9f9f9;
}

li span {
  margin: 0 0.5rem;
  color: #000; 
}

p {
  font-weight: bold;
  color: #333;
}
</style>
