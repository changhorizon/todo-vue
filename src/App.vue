<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import TodoList from './TodoList.vue'

const newTodo = ref('')
const todos = ref([])

// 引用输入框 DOM
const inputRef = ref(null)

const addTodo = () => {
  if (newTodo.value.trim() !== '') {
    todos.value.push({ text: newTodo.value, done: false })
    newTodo.value = ''
    inputRef.value.focus() // 添加任务后自动聚焦
  }
}

// 删除任务
const removeTodo = (index) => {
  todos.value.splice(index, 1)
}

// 切换完成状态
const toggleTodo = (index) => {
  todos.value[index].done = !todos.value[index].done
}

const clearCompleted = () => {
  todos.value = todos.value.filter((t) => !t.done)
}

// 计算属性：未完成任务数
const remaining = computed(() => todos.value.filter((t) => !t.done).length)

// 第一次渲染完成后也让输入框聚焦
onMounted(() => {
  inputRef.value.focus()
})

// 监听 todos 保存到本地
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
  <div class="todo-app">
    <h1>Todo List</h1>

    <input v-model="newTodo" placeholder="请输入任务" ref="inputRef" />
    <button @click="addTodo">添加</button>

    <TodoList
      :todos="todos"
      @toggle="toggleTodo"
      @remove="removeTodo"
      @clearCompleted="clearCompleted"
    />

    <p>还有{{ remaining }}个任务未完成</p>
  </div>
</template>

<style>
.todo-app {
  font-family: sans-serif;
  width: 320px;
  margin: 30px auto;
}
</style>
