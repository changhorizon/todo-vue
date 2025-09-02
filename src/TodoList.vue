<script setup>
import { defineProps, defineEmits } from 'vue'
import TodoItem from './TodoItem.vue'

// 父组件传来的 props
const { todos } = defineProps({
  todos: Array,
})

// 父组件事件透传
const emit = defineEmits(['toggle', 'remove', 'clearCompleted'])
</script>

<template>
  <!-- 条件渲染：空状态 -->
  <p v-if="!todos || todos.length === 0">暂无任务，请添加一个吧！</p>
  <ul v-else>
    <TodoItem
      v-for="(todo, index) in todos"
      :key="index"
      :todo="todo"
      :index="index"
      @toggle="emit('toggle', $event)"
      @remove="emit('remove', $event)"
    />
  </ul>

  <button v-if="todos.some((t) => t.done)" @click="emit('clearCompleted')">清空已完成任务</button>
</template>
