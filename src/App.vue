<script setup>
import { ref, onMounted, watch, computed } from "vue";
import { Icon } from "@iconify/vue";
// onmounted is kinda similar to useeffect with empty dependencies array, cuz it runs only once
// ref helps handling state, kinda similar to useref
// computed: for making a list, kinda similar to usememo, cuz we update the todo whenever any dependency changes
// watch: for watching any changes and performing side effects, kinda similar to useeffect with dependecy array

const todos = ref([]);
const name = ref("");

const input_content = ref("");
const input_category = ref(null);

const todos_asc = computed(() =>
  // sort by time created here
  todos.value.sort((a, b) => {
    return b.createdAt - a.createdAt;
  }),
);

const addTodo = () => {
  if (input_content.value.trim() === "" || input_category === null) {
    return;
  }

  todos.value.push({
    content: input_content.value,
    category: input_category.value,
    done: false,
    createdAt: new Date().getTime(),
  });
  input_content.value = "";
  input_category.value = null;
};

const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t != todo);
};

// if any new value, set it to local storage, works sorta like useeffect
watch(
  todos,
  (newValue) => {
    localStorage.setItem("todos", JSON.stringify(newValue));
  },
  { deep: true }, // deep true means that if anything changes inside todos, deep will catch it and call watch again
);

// set name to localstorage when any changes
watch(name, (newValue) => {
  localStorage.setItem("name", newValue);
});

onMounted(() => {
  name.value = localStorage.getItem("name") || "";
  todos.value = JSON.parse(localStorage.getItem("todos")) || [];
});
</script>

<template>
  <main class="app">
    <div class="container mx-auto p-4">
      <section class="greeting mb-8">
        <h2 class="flex items-center font-bold text-2xl">
          What's up,
          <input
            type="text"
            placeholder="your name here.."
            v-model="name"
            class="ml-1 flex-1 min-w-0 text-dark focus:outline-none"
          />
        </h2>
      </section>

      <section class="create-todo">
        <h3 class="font-bold text-lg mb-4">CREATE A TODO</h3>

        <form @submit.prevent="addTodo">
          <h4 class="text-gray-400 font-light mb-2">
            What's on your todo list
          </h4>
          <input
            type="text"
            placeholder="e.g. bake a cake"
            v-model="input_content"
            class="block text-lg py-4 px-6 rounded-lg shadow mb-6 focus:outline-none"
          />

          <h4 class="text-gray-400 font-light mb-2">Pick a category</h4>

          <div class="options flex gap-6 mb-6">
            <label
              class="flex flex-col items-center justify-center p-8 bg-white rounded-lg shadow cursor-pointer"
            >
              <input
                type="radio"
                name="category"
                value="business"
                v-model="input_category"
              />
              <span class="bubble business"></span>
              <div class="text-lg mt-2">Business</div>
            </label>

            <label
              class="flex flex-col items-center justify-center p-8 bg-white rounded-lg shadow cursor-pointer"
            >
              <input
                type="radio"
                name="category"
                value="personal"
                v-model="input_category"
              />
              <span class="bubble personal"></span>
              <div class="text-lg mt-2">Personal</div>
            </label>
          </div>
          <button
            type="submit"
            class="block py-4 px-6 bg-pink-500 text-white font-bold rounded-lg shadow hover:bg-pink-600 focus:outline-none"
          >
            Add todo
          </button>
        </form>
      </section>

      <section class="todo-list">
        <h3 class="font-bold text-lg my-4">TODO LIST</h3>
        <div class="list">
          <div
            v-for="todo in todos_asc"
            :key="todo.id"
            class="todo-item p-4 bg-white rounded-lg shadow mb-4 flex items-center gap-4 justify-between"
          >
            <div class="todo-content">
              <label class="mx-3">
                <input type="checkbox" v-model="todo.done" />
                <span :class="`bubble ${todo.category}`"></span>
              </label>

              <input
                :class="`ml-1 flex-1 min-w-0 text-dark focus:outline-none ${
                  todo.done == true ? 'line-through' : ''
                }`"
                type="text"
                v-model="todo.content"
              />
            </div>

            <div class="actions flex items-center">
              <div class="relative">
                <select
                  v-model="todo.category"
                  class="block appearance-none w-full bg-white border border-gray-400 hover:border-gray-500 p-3 pr-8 rounded-xl shadow leading-tight focus:outline-none focus:shadow-outline"
                >
                  <option value="business">Business</option>
                  <option value="personal">Personal</option>
                </select>
                <div
                  class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-gray-700"
                >
                  <Icon icon="fluent:chevron-down-16-regular" />
                </div>
              </div>
              <button
                class="bg-red-500 ml-3 p-3 text-white rounded-xl hover:bg-red-400"
                @click="removeTodo(todo)"
              >
                Delete
              </button>
            </div>
          </div>
        </div>
      </section>
    </div>
  </main>
</template>
