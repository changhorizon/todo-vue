<script setup>
// ================================
// 导入 Vue 提供的 API 与子组件
// ================================
import { toRefs } from 'vue'
import TodoItem from './TodoItem.vue'

// ================================
// 父组件传来的 props
// ================================
const props = defineProps({
  todos: {
    type: Array,
    required: true, // 必须传递 todos 数组
  },
})

// 保持响应式引用
const { todos } = toRefs(props)

// ================================
// 定义事件向父组件透传
// toggle: 切换完成状态
// remove: 删除任务
// update: 编辑任务后的更新
// ================================
const emit = defineEmits(['toggle', 'remove', 'update'])
</script>

<template>
  <!-- 任务列表容器 -->
  <div class="todo-list">
    <!-- 如果有任务就渲染 -->
    <ul v-if="todos.length > 0">
      <!-- 使用 transition-group 添加列表动画效果 -->
      <transition-group name="list" tag="ul">
        <!-- 遍历 todos 数组渲染每个任务项 -->
        <TodoItem
          v-for="todo in todos"
          :key="todo.id"
          :todo="todo"
          @toggle="emit('toggle', $event)"
          @remove="emit('remove', $event)"
          @update="emit('update', $event)"
        />
      </transition-group>
    </ul>
    <!-- 如果没有任务显示提示文字 -->
    <p v-else class="empty">暂无任务 🎉</p>
  </div>
</template>

<style scoped>
/* ================================
   列表动画效果
   enter-active / leave-active: 动画持续时间
   enter-from / leave-to: 初始和结束状态
   ================================ */
.list-enter-active,
.list-leave-active {
  transition: all 0.3s;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

/* ================================
   列表样式
   ================================ */
.todo-list {
  padding: 0 12px;
}
.todo-list ul {
  list-style: none;
  margin: 0 0 4px 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

/* ================================
   空列表提示样式
   ================================ */
.empty {
  text-align: center;
  color: var(--color-text-muted);
  padding: 16px;
  font-size: 0.9rem;
}
</style>
