<script setup>
import { ref } from 'vue'

const props = defineProps({
  todo: {
    type: Object,
    required: true,
  },
})

const emit = defineEmits(['toggle', 'remove', 'update'])

const editing = ref(false)
const editText = ref('')

// 切换完成状态
const onToggle = () => {
  emit('toggle', props.todo)
}

// 删除任务
const onRemove = () => {
  emit('remove', props.todo)
}

// 进入编辑模式
const onEdit = () => {
  editing.value = true
  editText.value = props.todo.text
}

// 保存编辑
const onSave = () => {
  const text = editText.value.trim()
  if (text && text !== props.todo.text) {
    emit('update', { ...props.todo, text })
  }
  editing.value = false
}

// 取消编辑
const onCancel = () => {
  editing.value = false
  editText.value = props.todo.text
}
</script>

<template>
  <li :class="['todo-item', { done: todo.done }]">
    <!-- 查看模式 -->
    <div v-if="!editing" class="view">
      <label @dblclick="onEdit" class="todo-label">
        <input type="checkbox" :checked="todo.done" @change="onToggle" />
        <span>{{ todo.text }}</span>
      </label>
      <button class="delete-btn" @click="onRemove">删除</button>
    </div>

    <!-- 编辑模式 -->
    <input
      v-else
      v-model="editText"
      class="edit-input"
      @keyup.enter="onSave"
      @keyup.esc="onCancel"
      @blur="onSave"
      autofocus
    />
  </li>
</template>

<style scoped>
.todo-item {
  display: flex;
  flex-direction: column;
  padding: 8px 12px;
  border-radius: var(--border-radius);
  background-color: var(--color-bg-list);
  box-shadow: var(--shadow-light);
  width: 100%;
  box-sizing: border-box;
}

/* 查看模式 */
.view {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  min-height: 36px; /* 保持与输入框一致 */
}

/* checkbox + 文字容器 */
.todo-label {
  display: flex;
  align-items: center;
  flex: 1;
  cursor: default;
}

/* checkbox 样式 */
.todo-label input[type='checkbox'] {
  margin-right: 8px;
  cursor: pointer;
}

/* 文字样式 */
.todo-label span {
  user-select: none;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  flex: 1;
  line-height: 1.3em; /* 汉字数字高度统一 */
}

/* 已完成任务删除线 */
.todo-item.done .todo-label span {
  text-decoration: line-through;
  color: #999;
}

/* 删除按钮 */
.delete-btn {
  flex-shrink: 0;
  background: transparent;
  border: none;
  color: var(--color-danger);
  cursor: pointer;
  font-size: 0.9rem;
  padding: 2px 6px;
  border-radius: 3px;
  transition:
    background 0.2s,
    color 0.2s;
}
.delete-btn:hover {
  background-color: rgba(255, 0, 0, 0.1);
  color: var(--color-danger-hover);
}

/* 编辑模式 */
.edit-input {
  width: 100%;
  padding: 6px 8px;
  font-size: 1rem;
  border: 1px solid var(--color-border);
  border-radius: 3px;
  outline: none;
  box-sizing: border-box;
  min-height: 36px; /* 与查看模式一致 */
  line-height: 1.3em;
}
</style>
