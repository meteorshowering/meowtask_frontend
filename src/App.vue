<template>
  <div class="container">
    <button @click="toggleLeftColumn" class="toggle-button">弹出</button>
    <div v-if="leftColumnOpen" class="overlay" @click="toggleLeftColumn"></div>
    <div class="left-column" :class="{ 'open': leftColumnOpen }">
      <h1>MeowTask</h1>
      <div class="form-group">
        <label for="newTask">任务内容：</label>
        <input v-model="newTask" placeholder="输入新任务" @keyup.enter="addTask" class="rectangle-2"/>
      </div>
      <div class="form-group">
        <label for="newTaskTime">任务时间：</label>
        <input v-model="newTaskTime" type="datetime-local" placeholder="设置任务时间" @keyup.enter="addTask" class="rectangle-3"/>
      </div>
      <div class="form-group">
        <label for="newTaskTag">任务标签：</label>
        <input v-model="newTaskTag" placeholder="添加标签" @keyup.enter="addTask" class="rectangle-4"/>
      </div>
      <div class="form-group">
        <label for="newTaskPoints">任务积分：</label>
        <input v-model.number="newTaskPoints" type="number" placeholder="积分（默认2）" @keyup.enter="addTask" class="rectangle-5"/>
      </div>
        <button @click="addTask" class="ellipse">添加任务</button>
    </div>
    <div class="right-column">
      <div class="fish-container">
        <img v-for="index in 6" :key="index" src="@/assets/fish.jpg" alt="小鱼" class="fish-img">
      </div>
      <div class="task-list">
        <TaskItem 
          v-for="(task, index) in filteredTasks"
          :key="task.id" 
          :task="task" 
          @complete="handleTaskComplete"
        />
      </div>
      <p class="cat-text">请投喂猫</p>
      <img src="@/assets/cat.jpg" alt="等饭猫猫" class="bottom-cat-img">
    </div>
  </div>
</template>
          <!-- v-for="task in filteredTasks"  -->

<script setup>
import { ref, computed,onMounted ,onBeforeUnmount  } from 'vue';
import axios from 'axios';
const tasks = ref([]); //这是从后端读取的，对接的话要打开
const newTask = ref('');
import TaskItem from './components/TaskItem.vue';
const newTaskTime = ref('');
const newTaskTag = ref('');
const newTaskPoints = ref(2);

// 任务数据
// const tasks = ref([
//   { id: 1, content: '完成智能计算系统大作业', date: '2025/05/06', completed: false },
//   { id: 2, content: '慢跑30分钟', date: '2025/05/06', completed: false },
//   { id: 3, content: '阅读30分钟', date: '2025/05/07', completed: false },
//   { id: 4, content: '购买猫粮', date: '2025/05/08', completed: false },
// ]);
//从后端读取,这个是读一次的，可以先试试这个是不是可以的
// const fetchTasks = async () => {
//   try {
//     const response = await axios.get('/api/tasks')
//     tasks.value = response.data
//   } catch (error) {
//     console.error('获取任务列表失败:', error)
//   }
// }
// onMounted(() => {
//   fetchTasks()
// })

//下面是socket对接的，已经试过，效果良好。可以读取
let socket = null

onMounted(() => {
  // 建立 WebSocket 连接
  socket = new WebSocket('ws://localhost:8080/ws')

  // 连接建立成功
  socket.onopen = () => {
    console.log('WebSocket 已连接')
  }

  // 接收到后端推送的数据
  socket.onmessage = (event) => {
    const newTaskList = JSON.parse(event.data)
    tasks.value = newTaskList
  }

  // 连接关闭
  socket.onclose = () => {
    console.log('WebSocket 已关闭')
  }

  // 出错
  socket.onerror = (error) => {
    console.error('WebSocket 错误:', error)
  }
})

// 页面卸载前关闭 WebSocket
onBeforeUnmount(() => {
  if (socket) {
    socket.close()
  }
})

// let nextTaskId = ref(5);
const addTask = () => {
  if (newTask.value.trim()) {
    tasks.value.push({
      id: Date.now(), // 前端先临时生成个 id
      content: newTask.value.trim(),
      date: newTaskTime.value,
      // tag: newTaskTag.value,
      // points: parseInt(newTaskPoints.value) || 2,
      completed: false
    });
      // 发送给后端
    if (socket && socket.readyState === WebSocket.OPEN) {
      socket.send(JSON.stringify({
        action: 'create_task',
        data: newTask
      }))
    } else {
      console.error('WebSocket 未连接')
    }
    newTask.value = '';
    newTaskTime.value = '';
    newTaskTag.value = '';
    newTaskPoints.value = 2;
  }
};
// 计算属性：过滤已完成的任务
const filteredTasks = computed(() => 
  tasks.value.filter(task => !task.completed)
);

// // 处理任务完成
// const handleTaskComplete = (taskId) => {
//   const taskIndex = tasks.value.findIndex(task => task.id === taskId);
//   if (taskIndex !== -1) {
//     tasks.value[taskIndex].completed = true;
//   }
// };
// 处理任务完成
const handleTaskComplete = (taskId) => {
  const taskIndex = tasks.value.findIndex(task => task.id === taskId)
  if (taskIndex !== -1) {
    // 本地更新状态
    tasks.value[taskIndex].completed = true

    // 推送给后端
    if (socket && socket.readyState === WebSocket.OPEN) {
      socket.send(JSON.stringify({
        action: 'complete_task',
        data: { id: taskId }
      }))
    } else {
      console.error('WebSocket 未连接')
    }
  }
}

const leftColumnOpen = ref(false);

const toggleLeftColumn = () => {
  leftColumnOpen.value = !leftColumnOpen.value;
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: row; 
  width: 880px;
  /* max-width: 1200px; */
  margin: 0;
  background-color: aquamarine;
}

.toggle-button {
  position: fixed;
  left: 20px;
  top: 20px;
  z-index: 1000;
}

.left-column {
  position: relative;
  left: -50%;
  top: 0;
  height: 100%;
  transition: left 0.3s ease;
  z-index: 20;
  background-color: aliceblue;
}
.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.left-column.open {
  left: 0;
}

.right-column {
  width: 100%;
  flex:1;
  height: 100%;
  background-color: #fff;
  padding: 20px;
  transition: margin-left 0.3s ease; 
}

.left-column.open + .right-column {
  margin-left: 0%; 
}

.fish-container {
  display: flex;
  justify-content: space-around;
  margin-bottom: 20px;
}
.rectangle-2 { border: 5px solid; border-color: #202020; border-radius: 10px; }
.rectangle-3 { border: 5px solid; border-color: #202020; border-radius: 10px; } 
.rectangle-4 { border: 5px solid; border-color: #202020; border-radius: 10px; } 
.rectangle-5 { border: 5px solid; border-color: #202020; border-radius: 10px; } 
.rectangle-6 { border: 5px solid; border-color: #202020; border-radius: 10px; } 
.ellipse { border: 5px solid; border-color: #202020; border-radius: 41.5px / 40.5px; }

.fish-img {
  width: 50px;
  height: 50px;
  object-fit: contain;
}

.task-list {
  list-style-type: none;
  /* padding: 0; */
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 20px;
  overflow-y: auto;
}

.cat-text {
  text-align: right;
  font-size: 20px;
  margin: 20px 0;
}

.bottom-cat-img {
  width: 150px;
  float: right;
  border-radius: 8px;
}
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5); /* 半透明黑 */
  z-index: 10;
}

</style>