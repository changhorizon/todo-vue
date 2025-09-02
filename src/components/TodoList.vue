<script setup>
import { defineProps, defineEmits } from 'vue'
import TodoItem from './TodoItem.vue'

// 父组件传来的 props
const { todos } = defineProps({
  todos: Array,
})

// 父组件事件透传
const emit = defineEmits(['toggle', 'remove'])
</script>

<template>
  <div class="todo-list">
    <ul v-if="todos && todos.length > 0">
      <TodoItem
        v-for="(todo, index) in todos"
        :key="index"
        :todo="todo"
        :index="index"
        @toggle="emit('toggle', $event)"
        @remove="emit('remove', $event)"
      />
    </ul>
  </div>
</template>

<style scoped>
.todo-list {
  padding: 0 12px;
}
.todo-list ul {
  list-style: none;
  margin: 0 0 4px 0; /* 下方增加间距 */
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.todo-list li {
  border-radius: var(--border-radius);
  padding: 8px;
  background-color: var(--color-bg-list);
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: var(--shadow-light);
}

.todo-list li.done {
  text-decoration: line-through;
  color: #999;
}

.todo-list li input[type='checkbox'] {
  margin-right: 8px;
}
</style>
