<template>
  <div class="task-item">
    <img 
      src="@/assets/xian.png" 
      alt="毛线" 
      class="button" 
      @click="completeTask"
    >
    {{ task.content }}
    <span class="date">{{ task.date }}</span>
  </div>
</template>

<script setup>
const props = defineProps({ task: { type: Object, required: true } });

const emit = defineEmits(['complete']);

const completeTask = () => {
  // emit('complete', props.task.id);
  props.task.completed = true;
  emit('complete', props.task.id);
};

// const completeTask = (index) => {
//   tasks.value[index].completed = !tasks.value[index].completed;
// };
// const totalPoints = computed(() => {
//   return tasks.value.filter(task => task.completed).reduce((sum, task) => sum + task.points, 0);
// });

</script>

<style scoped>
.task-item {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
  padding: 10px;
  border-radius: 4px;
  background-color: #bbbbbb;
  color: #000000;
  transition: all 0.3s ease;
  width: 100%;
  box-sizing: border-box;
}

.task-item:hover {
  background-color: #C8CFE9;
  transform: translateX(5px);
}

.button {
  width: 24px;
  height: 24px;
  margin-right: 15px;
  cursor: pointer;
  transition: transform 0.2s;
}

.button:hover {
  transform: scale(1.1);
}

.date {
  margin-left: auto;
  color: #888;
  font-size: 14px;
}

</style>