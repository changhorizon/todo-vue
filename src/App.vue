<script setup>
// ================================
// 导入 Vue 提供的组合式 API
// ================================
import { ref, onMounted, computed, watch } from 'vue'
import TodoList from './components/TodoList.vue'

// ================================
// 响应式数据
// ================================
const newTodo = ref('') // 输入框的文本
const todos = ref([]) // 存储任务数组
const filter = ref('all') // 当前过滤状态：all / active / completed
const isError = ref(false) // 输入错误状态（空文本时触发抖动）
const inputRef = ref(null) // 输入框 DOM 引用，用于聚焦

// ================================
// 任务操作方法
// ================================
const addTodo = () => {
  if (newTodo.value.trim() === '') {
    inputRef.value.focus()
    isError.value = true
    setTimeout(() => (isError.value = false), 200)
    return
  }
  todos.value.push({
    id: Date.now(),
    text: newTodo.value,
    done: false,
  })
  newTodo.value = ''
  inputRef.value.focus()
}

const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t.id !== todo.id)
}

const toggleTodo = (todo) => {
  todo.done = !todo.done
}

const updateTodo = (updated) => {
  todos.value = todos.value.map((t) => (t.id === updated.id ? updated : t))
}

const clearCompleted = () => {
  todos.value = todos.value.filter((t) => !t.done)
}

// ================================
// 计算属性
// ================================
const remaining = computed(() => todos.value.filter((t) => !t.done).length)
const hasCompleted = computed(() => todos.value.some((t) => t.done))

const filteredTodos = computed(() => {
  let list = todos.value
  if (filter.value === 'active') list = list.filter((t) => !t.done)
  else if (filter.value === 'completed') list = list.filter((t) => t.done)
  return list
})

// ================================
// 主题功能
// ================================
const isDark = ref(localStorage.getItem('isDark') === 'true')
const applyTheme = (dark) => {
  document.body.classList.toggle('dark-theme', dark)
  localStorage.setItem('isDark', dark)
}

onMounted(() => {
  inputRef.value.focus()
  applyTheme(isDark.value)
})

watch(isDark, (val) => applyTheme(val))

// ================================
// 本地存储 todos
// ================================
const saved = localStorage.getItem('todos')
if (saved) todos.value = JSON.parse(saved)

watch(
  todos,
  (newValue) => {
    localStorage.setItem('todos', JSON.stringify(newValue))
  },
  { deep: true },
)
</script>

<template>
  <!-- 主容器 -->
  <div class="todo-app">
    <!-- 头部：标题 + 主题切换 -->
    <header class="header">
      <h1>Todo List</h1>
      <button class="theme-toggle" @click="isDark = !isDark">
        {{ isDark ? '☀️' : '🌙' }}
      </button>
    </header>

    <!-- 筛选按钮 -->
    <section class="filters">
      <button @click="filter = 'all'" :class="{ active: filter === 'all' }">全部</button>
      <button @click="filter = 'active'" :class="{ active: filter === 'active' }">未完成</button>
      <button @click="filter = 'completed'" :class="{ active: filter === 'completed' }">
        已完成
      </button>
    </section>

    <!-- 任务列表和底部信息 -->
    <div class="list-container">
      <TodoList
        :todos="filteredTodos"
        @toggle="toggleTodo"
        @remove="removeTodo"
        @update="updateTodo"
      />
      <footer class="footer">
        <span v-if="filter !== 'completed'" class="remaining">
          还有 {{ remaining }} 个任务未完成
        </span>
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

    <!-- 底部输入区 -->
    <section class="todo-input">
      <input
        v-model="newTodo"
        :class="{ 'input-error': isError }"
        placeholder="请输入任务"
        ref="inputRef"
        @keyup.enter="addTodo"
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
