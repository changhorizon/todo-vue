<script setup>
import { defineProps, defineEmits } from 'vue'
import TodoItem from './TodoItem.vue'

// 父组件传来的 props
const props = defineProps({
  todos: Array,
})

// 父组件事件透传
const emit = defineEmits(['toggle', 'remove'])
</script>

<template>
  <ul>
    <TodoItem
      v-for="(todo, index) in todos"
      :key="index"
      :todo="todo"
      :index="index"
      @toggle="emit('toggle', $event)"
      @remove="emit('remove', $event)"
    />
  </ul>
</template>
