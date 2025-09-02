<script setup>
import { defineProps, defineEmits } from 'vue'

// 父组件传来的 props
const props = defineProps({
  todo: Object,
  index: Number,
})

const emit = defineEmits(['toggle', 'remove'])

// 子组件要触发的事件
const onToggle = () => {
  emit('toggle', props.index)
}

const onRemove = () => {
  emit('remove', props.index)
}
</script>

<template>
  <li :class="['todo-item', { done: todo.done }]">
    <label>
      <input type="checkbox" :checked="todo.done" @change="onToggle" />
      <span>{{ todo.text }}</span>
    </label>
    <button class="delete-btn" @click="onRemove">删除</button>
  </li>
</template>

<style scoped>
.todo-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
  border-radius: 3px;
  background-color: var(--color-bg-list);
  box-shadow: var(--shadow-light);
}

.todo-item.done span {
  text-decoration: line-through;
  color: #999;
}

.todo-item label {
  display: flex; /* 保持 checkbox 和文字垂直居中 */
  align-items: center;
  cursor: pointer;
}

.todo-item input[type='checkbox'] {
  margin-right: 8px;
}

.delete-btn {
  border: none;
  background: transparent;
  color: #c00;
  cursor: pointer;
  font-size: 0.9rem;
}
</style>
