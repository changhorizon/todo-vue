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
  <li :class="{ done: todo.done }">
    <input type="checkbox" :checked="todo.done" @change="onToggle" />
    {{ todo.text }}
    <button @click="onRemove">删除</button>
  </li>
</template>

<style scoped>
.done {
  text-decoration: line-through;
  color: gray;
}
</style>
