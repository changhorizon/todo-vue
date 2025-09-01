<script setup>
import { ref, computed, watch } from 'vue'

const newTodo = ref('')
const todos = ref([
  { text: '学习 Vue', done: false },
  { text: '写一个 TodoList', done: true },
])

const addTodo = () => {
  if (newTodo.value.trim() !== '') {
    todos.value.push({ text: newTodo.value, done: false })
    newTodo.value = ''
  }
}

const removeTodo = (index) => {
  todos.value.splice(index, 1)
}

const toggleTodo = (index) => {
  todos.value[index].done = !todos.value[index].done
}

// 计算属性
const remaining = computed(() => todos.value.filter((t) => !t.done).length)

// 监听 todos 的变化
watch(
  todos,
  (newValue) => {
    localStorage.setItem('todos', JSON.stringify(newValue))
  },
  { deep: true }, // 因为 todos 是数组对象，需要深度监听
)

// 初始化时读取本地数据
const saved = localStorage.getItem('todos')
if (saved) {
  todos.value = JSON.parse(saved)
}
</script>

<template>
  <h1>Todo List</h1>

  <input v-model="newTodo" placeholder="请输入任务" />
  <button @click="addTodo">添加</button>

  <ul>
    <li
      v-for="(todo, index) in todos"
      :key="index"
      :style="{ textDecoration: todo.done ? 'line-through' : 'none' }"
    >
      <input type="checkbox" :checked="todo.done" @change="toggleTodo(index)" />
      {{ todo.text }}
      <button @click="removeTodo(index)">删除</button>
    </li>
  </ul>
  <p>还有{{ remaining }}个任务未完成</p>
</template>

<style>
.todo-app {
  font-family: sans-serif;
  width: 300px;
  margin: 30px auto;
}
</style>
