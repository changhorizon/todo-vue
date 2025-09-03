<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import TodoList from './components/TodoList.vue'

const newTodo = ref('')
const todos = ref([])
const filter = ref('all')
const isError = ref(false)
const inputRef = ref(null)

const addTodo = () => {
  if (newTodo.value.trim() === '') {
    inputRef.value.focus()
    isError.value = true
    setTimeout(() => {
      isError.value = false
    }, 200)
    return
  }
  isError.value = false
  todos.value.push({ text: newTodo.value, done: false })
  newTodo.value = ''
  inputRef.value.focus()
}

const removeTodo = (index) => {
  todos.value.splice(index, 1)
}

const toggleTodo = (index) => {
  todos.value[index].done = !todos.value[index].done
}

const clearCompleted = () => {
  todos.value = todos.value.filter((t) => !t.done)
}

const remaining = computed(() => todos.value.filter((t) => !t.done).length)
const hasCompleted = computed(() => todos.value.some((t) => t.done))

const filteredTodos = computed(() => {
  let list = todos.value
  if (filter.value === 'active') list = list.filter((t) => !t.done)
  else if (filter.value === 'completed') list = list.filter((t) => t.done)
  return list
})

// 从 localStorage 里取主题
const isDark = ref(localStorage.getItem('isDark') === 'true')

// 封装一个函数，专门负责应用主题和保存
const applyTheme = (dark) => {
  if (dark) {
    document.body.classList.add('dark-theme')
  } else {
    document.body.classList.remove('dark-theme')
  }
  localStorage.setItem('isDark', dark) // 保存到 localStorage
}

onMounted(() => {
  inputRef.value.focus()

  // 页面加载时，执行一次初始化
  applyTheme(isDark.value)
})

watch(isDark, (val) => {
  applyTheme(val)
})

const saved = localStorage.getItem('todos')
if (saved) {
  todos.value = JSON.parse(saved)
}

watch(
  todos,
  (newValue) => {
    localStorage.setItem('todos', JSON.stringify(newValue))
  },
  { deep: true },
)
</script>

<template>
  <div class="todo-app">
    <header class="header">
      <h1>Todo List</h1>
      <button class="theme-toggle" @click="isDark = !isDark">
        {{ isDark ? '☀️' : '🌙' }}
      </button>
    </header>

    <section class="filters">
      <button @click="filter = 'all'" :class="{ active: filter === 'all' }">全部</button>
      <button @click="filter = 'active'" :class="{ active: filter === 'active' }">未完成</button>
      <button @click="filter = 'completed'" :class="{ active: filter === 'completed' }">
        已完成
      </button>
    </section>

    <div class="list-container">
      <TodoList
        :todos="filteredTodos"
        @toggle="toggleTodo"
        @remove="removeTodo"
        @clearCompleted="clearCompleted"
      />
      <footer class="footer">
        <span v-if="filter !== 'completed'" class="remaining"
          >还有 {{ remaining }} 个任务未完成</span
        >
        <span v-else></span>
        <button
          v-if="hasCompleted && filter !== 'active'"
          class="clear-btn"
          @click="clearCompleted"
        >
          清空已完成任务
        </button>
      </footer>
    </div>

    <section class="todo-input">
      <input
        v-model="newTodo"
        :class="{ 'input-error': isError }"
        placeholder="请输入任务"
        ref="inputRef"
      />
      <button @click="addTodo">添加</button>
    </section>
  </div>
</template>

<style src="./styles/global.css"></style>

<style scoped>
/* 整体布局 */
.todo-app {
  display: flex;
  flex-direction: column;
  height: 100vh;
  max-width: 480px;
  min-width: 280px;
  margin: 0 auto;
  background: var(--color-bg);
  border-radius: var(--border-radius);
  box-shadow: var(--shadow-medium);
}

/* 头部全屏背景 */
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 480px; /* 限制宽度 */
  margin: 0 auto; /* 居中 */
  padding: 0 16px;
  background-color: var(--color-header);
  box-shadow: var(--shadow-light);
  height: 60px;
  width: 100%;
}
.header h1 {
  font-size: 1.5rem;
  margin: 0;
  text-transform: uppercase;
  color: white;
}
.theme-toggle {
  background: transparent;
  border: none;
  color: white;
  font-size: 1.2rem;
  cursor: pointer;
  padding: 4px 8px;
  border-radius: 4px;
  transition: background 0.2s;
}
.theme-toggle:hover {
  background: rgba(255, 255, 255, 0.2);
}

/* 筛选按钮 */
.filters {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  justify-content: center;
  margin: 12px 0;
  padding: 0 12px;
}
.filters button {
  flex: 1 1 30%;
  padding: 6px 0;
  border: 1px solid var(--color-border);
  border-radius: var(--border-radius);
  color: var(--color-filter-text);
  background: white;
  cursor: pointer;
  transition: all 0.2s;
}
.dark-theme .filters button {
  background: #2c2c2c; /* 夜间背景 */
}

.filters button:hover {
  color: var(--color-filter-text-hover);
}
.filters button.active {
  background-color: var(--color-primary);
  color: white;
}

/* 列表容器 */
.list-container {
  flex: 1;
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding-bottom: 72px; /* 避免被底部输入区遮挡 */
}

/* 底部信息与清理按钮同一行 */
.footer {
  text-align: left;
  color: var(--color-text-muted);
  font-size: 0.875rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 12px;
  min-height: 31px;
}
.footer button {
  border: none;
  background: transparent;
  color: var(--color-danger);
  cursor: pointer;
  font-size: 0.875rem;
}
.footer .clear-btn {
  background-color: var(--color-danger);
  color: white;
  padding: 6px 12px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background 0.2s;
}
.footer .clear-btn:hover {
  background-color: var(--color-danger-hover);
}

/* 输入区固定底部全屏背景 */
.todo-input {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  max-width: 480px;
  padding: 12px;
  background-color: var(--color-input);
  display: flex;
  gap: 8px;
  z-index: 10;
  box-sizing: border-box;
}

.dark-theme .todo-input {
  border-top: 1px solid var(--color-border);
  box-shadow: 0 -2px 6px rgba(0, 0, 0, 0.4);
}

/* 输入框与按钮 */
.todo-input input,
.todo-input button {
  box-sizing: border-box;
  height: 36px;
  line-height: 36px;
  padding: 0 12px;
  font-size: 1rem;
  border-radius: 3px;
  border: 1px solid var(--color-border);
}
.todo-input button {
  background-color: var(--color-primary);
  color: white;
  border: none;
  cursor: pointer;
  transition: background 0.2s;
}
.todo-input input {
  flex: 1 1 auto;
  min-width: 0;
}
.todo-input button:hover {
  background-color: var(--color-primary-hover);
}

.todo-input input:focus {
  outline: none; /* 去掉默认蓝色焦点线 */
  border-color: var(--color-primary); /* 可选：聚焦时边框高亮 */
  box-shadow: 0 0 0 2px rgba(51, 153, 255, 0.3); /* 可选：淡蓝色光晕代替默认白圈 */
}

/* 极窄屏竖排 */
@media (max-width: 350px) {
  .todo-input {
    display: block;
    height: auto;
    padding: 8px 16px;
  }
  .todo-input input,
  .todo-input button {
    display: block;
    width: 100%;
    margin-bottom: 6px;
  }
  .todo-input button {
    margin-bottom: 0;
  }
}

/* 输入框错误抖动 */
.input-error {
  border: 1px solid var(--color-danger);
  animation: shake 0.2s;
}
@keyframes shake {
  0% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-4px);
  }
  50% {
    transform: translateX(4px);
  }
  75% {
    transform: translateX(-4px);
  }
  100% {
    transform: translateX(0);
  }
}
</style>
