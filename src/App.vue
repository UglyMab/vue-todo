<template>
  <div class="container">
    <img src="./assets/image.svg" alt="" class="image" />
    <h2 class="title">Today I need to</h2>
    <form class="input-form" @submit.prevent="addTodo">
      <input
        type="text"
        class="input"
        v-model="inputVal"
        placeholder="Add new todo..."
      />
      <button type="submit" class="submit-btn" v-if="inputVal">Submit</button>
    </form>

    <draggable
      tag="ul"
      :list="filteredTodos"
      class="todo-list"
      item-key="id"
      handle=".handle"
    >
      <template #item="{ element }">
        <li class="todo-item">
          <img src="./assets/drag.svg" alt="" class="todo-drag handle" />
          <label :for="element.id" class="todo-label">
            <input
              type="checkbox"
              name="todo"
              :id="element.id"
              v-model="element.done"
              class="todo-cbx"
            />
            <span v-if="!element.edit" class="todo-title">{{
              element.title
            }}</span>
            <input
              class="todo-edit-input"
              @keyup.enter="editTodoSubmit(element)"
              @blur="editTodoSubmit(element)"
              v-autofocus
              v-if="element.edit"
              type="text"
              v-model="element.title"
            />
          </label>
          <img
            src="./assets/edit.svg"
            @click="editTodo(element)"
            alt=""
            class="todo-edit"
          />
          <img
            src="./assets/delete.svg"
            @click="removeTodo(element)"
            alt=""
            class="todo-remove"
          />
        </li>
      </template>
    </draggable>

    <div class="status-wrapper" v-if="todos.length">
      <div class="status">
        <p class="status-count">
          <span>{{ completedTodos }}</span> tasks
        </p>
        <p class="status-title">Completed</p>
        <span class="status-bar"
          ><span :style="{ width: completedBarWidth + '%' }"></span
        ></span>
      </div>
      <div class="status">
        <p class="status-count">
          <span>{{ uncompletedTodos }}</span>
          tasks
        </p>
        <p class="status-title">To be finished</p>
        <span class="status-bar status-bar--finished">
          <span :style="{ width: uncompletedBarWidth + '%' }"></span
        ></span>
      </div>
    </div>
    <div class="buttons-wrapper" v-if="todos.length">
      <button class="button" @click="checkAll" v-if="uncompletedTodos">
        Check All
      </button>
      <button
        :class="[{ active: filter == 'All' }, 'button']"
        @click="filterChange('All')"
      >
        All
      </button>
      <button
        v-if="completedTodos && uncompletedTodos"
        :class="[{ active: filter == false }, 'button']"
        @click="filterChange(false)"
      >
        Active
      </button>
      <button
        v-if="completedTodos && uncompletedTodos"
        :class="[{ active: filter == true }, 'button']"
        @click="filterChange(true)"
      >
        Complited
      </button>
      <button class="button" @click="clearCompleted" v-if="completedTodos">
        Clear complited
      </button>
    </div>
    <div v-if="!todos.length" class="message-wrapper">
      <img src="./assets/check.svg" alt="" />
      <p class="message">Congrat, you have no more tasks to do</p>
    </div>
  </div>
</template>

<script setup>
import draggable from 'vuedraggable'
import { ref, watch, onMounted, computed } from 'vue'

const todos = ref([])
const inputVal = ref('')
const filter = ref('All')
watch(
  todos,
  (newVal) => {
    localStorage.setItem('todos', JSON.stringify(newVal))
  },
  {
    deep: true,
  },
)

const addTodo = () => {
  if (inputVal.value.trim() === '') {
    return
  }
  todos.value.push({
    title: inputVal.value,
    done: false,
    editable: false,
    createdAt: new Date().getTime(),
  })
  inputVal.value = ''
}
const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t !== todo)
}
const editTodo = (todo) => {
  todo.edit = !todo.edit
}
const editTodoSubmit = (todo) => {
  todo.edit = false
}
const filterChange = (val) => {
  filter.value = val
}
const clearCompleted = () => {
  filter.value = 'All'
  todos.value = todos.value.filter((t) => t.done === false)
}
const checkAll = () => {
  filter.value = 'All'
  todos.value = todos.value.map((t) => {
    return { ...t, done: true }
  })
}
onMounted(() => {
  todos.value = JSON.parse(localStorage.getItem('todos')) || []
})
const filteredTodos = computed(() => {
  let f = filter.value
  return todos.value.filter(function (elem) {
    if (f === 'All') return true
    else return elem.done === f
  })
})
const completedTodos = computed(() =>
  todos.value.reduce(function (acc, item) {
    item.done ? acc++ : acc
    return acc
  }, 0),
)
const uncompletedTodos = computed(() =>
  todos.value.reduce((acc, item) => {
    item.done ? acc : acc++
    return acc
  }, 0),
)
const completedBarWidth = computed(
  () =>
    (100 / (uncompletedTodos.value + completedTodos.value)) *
    completedTodos.value,
)
const uncompletedBarWidth = computed(() => 100 - completedBarWidth.value)

const vAutofocus = {
  mounted: (el) => {
    el.focus()
  },
}
</script>
