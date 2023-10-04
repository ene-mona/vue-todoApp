<template>
  <header>
    <div class="wrapper">
      <form class="name" @submit.prevent="submit">
        <input
          ref="inp"
          type="text"
          placeholder="Enter your name"
          v-model="getName"
          :readonly="readname"
          @click="enterName"
          @input="handleNameInput"
        />
        <button v-if="!readname">Ok</button>
      </form>
      <!-- <h1 class="getName">{{ getName }}'s Todos</h1> -->
      <!-- <button v-if="hideInput" @click="hideInput = false">Change name</button> -->
      <nav>
        <!-- <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink> -->
      </nav>
      <div class="addTodo">
        <!-- <h4>Add a todo to your list</h4> -->
        <form @submit.prevent="addTodo" class="enterTodo">
          <input
            type="text"
            v-model="newTodo"
            placeholder="What will you like to do?"
          />

          <select v-model="setPriorities" required>
            <option value="important">Important</option>
            <option value="very-important">Very important</option>
          </select>
          <button>Add</button>
        </form>
      </div>
    </div>
  </header>

  <div class="todo-wrapper">
    <div class="todos" v-for="(getTodo, index) in getTodos">
      <input
        :class="`todos${index} ${getTodo.priority}`"
        type="text"
        :value="getTodo.todo"
        :readonly="readOnly"
        @input="handleInput($event, index)"
        @keyup.enter="handleEnterKey(index)"
        :style="{
          'text-decoration': getTodo.completed ? 'line-through' : 'none',
        }"
        required
      />
      <div class="crud">
        <button>
          <input
            class="completeBtn"
            type="checkbox"
            :checked="getTodo.completed"
            @change="handleCompleted(index)"
          />
        </button>
        <button class="deleteBtn" @click="deleteTodo(index)">
          <i class="bi bi-trash"></i>
        </button>
        <button class="editBtn" @click="editTodo(index)">
          <i class="bi bi-pen-fill"></i>
        </button>
      </div>
    </div>
  </div>

  <!-- <RouterView /> -->
</template>

<script setup>
import { onMounted, onUpdated, ref } from "vue";

// import { RouterLink, RouterView } from "vue-router";
// import HelloWorld from "./components/HelloWorld.vue";
// const name = ref("");
const getName = ref(null);
const newTodo = ref("");

const todoItems = ref([]);
// const completed = ref(false);
const getTodos = ref(null);
const readOnly = ref(true);
const readname = ref(false);

const setPriorities = ref("important");
const inp = ref(null);
const editInp = ref(null);
const editedTodo = ref(null);

const sortTodos = ref(null);

onMounted(() => {
  if (localStorage.getItem("name")) {
    if (localStorage.getItem("name").includes("'s todo ")) {
      getName.value = localStorage.getItem("name");
    } else {
      getName.value = localStorage.getItem("name") + "'s todo ";
    }
  }

  if (!getName.value) {
    inp.value.focus();
    readname.value = false;
    console.log(readname.value);
  } else {
    inp.value.blur();
    readname.value = true;
    console.log(readname.value);
  }
  getTodos.value = localStorage.getItem("todos");
  getTodos.value = getTodos.value ? JSON.parse(getTodos.value) : [];
  console.log("updated from mounted");
});
onUpdated(() => {
  // sortTodos.value = localStorage.getItem("todos");
  // sortTodos.value = sortTodos.value ? JSON.parse(sortTodos.value) : [];
  sortTodos.value = [...getTodos.value];

  console.log("updated");
  console.log(sortTodos.value);
  console.log(getTodos.value);
});

// METHODS

const handleNameInput = (e) => {
  getName.value = e.target.value;
  localStorage.setItem("name", getName.value);
};

const enterName = () => {
  inp.value.focus();
  readname.value = false;
};
const submit = () => {
  inp.value.blur();
  readname.value = true;

  if (getName.value && getName.value.includes("'s todo ")) {
    getName.value = getName.value;
  } else if (getName.value && !getName.value.includes("'s todo ")) {
    getName.value = getName.value + "'s todo ";
  } else {
    getName.value = "";
  }
  localStorage.setItem("name", getName.value);
};
const todoLogic = () => {
  const todos = {
    todo: newTodo.value,
    completed: false,
    priority: setPriorities.value,
  };

  let checkForMutipleTodos = getTodos.value.find(
    (todo, ind) => todo.todo === todos.todo
  );

  if (todos.todo && !checkForMutipleTodos) {
    getTodos.value.push(todos);

    const updatedData = JSON.stringify(getTodos.value);
    todoItems.value.push(todos);

    localStorage.setItem("todos", updatedData);
  }

  newTodo.value = "";
};

const handleCompleted = (index) => {
  getTodos.value[index].completed = !getTodos.value[index].completed;

  const updatedData = JSON.stringify(getTodos.value);
  localStorage.setItem("todos", updatedData);
};

const deleteTodo = (index) => {
  getTodos.value = getTodos.value.filter(
    (todo, ind) => todo !== getTodos.value[index]
  );
  const updatedData = JSON.stringify(getTodos.value);
  localStorage.setItem("todos", updatedData);
};

const editTodo = (index) => {
  if (getTodos.value[index].completed) {
    getTodos.value[index].completed = false;
    const updatedData = JSON.stringify(getTodos.value);
    localStorage.setItem("todos", updatedData);
  }
  editInp.value = document.querySelector(`.todos${index}`);
  editInp.value.readOnly = false;
  if (editInp.value) {
    editInp.value.focus();
  }
};
const handleEnterKey = (index) => {
  editInp.value = document.querySelector(`.todos${index}`);
  if (editInp.value.value !== "") {
    if (editedTodo.value !== null) {
      getTodos.value[index].todo = editedTodo.value;
      const updatedData = JSON.stringify(getTodos.value);
      localStorage.setItem("todos", updatedData);
      editInp.value.readOnly = true;
      editInp.value.blur();
    }
  }
};
const handleInput = (e, index) => {
  editedTodo.value = e.target.value;
};

const addTodo = () => {
  todoLogic();
};
</script>

<style scoped>
.wrapper {
  position: sticky !important;
  top: 0;
}
.name {
  width: 400px;
  position: relative;
  height: 40px;
  border-radius: 25px;
  margin-bottom: 20px;
}

.name input[type="text"] {
  width: 100%;
  height: 100%;
  border-radius: 25px;
  font-size: 23px;
  padding: 0 20px;
  text-transform: capitalize;
}

.getName {
  text-transform: capitalize;
  margin-bottom: 20px;
}
.name input[type="text"]:focus {
  outline: none;
}
.name button {
  position: absolute;
  top: 50%;
  right: 0;
  transform: translateY(-50%);
  border: none;
  padding: 7px;
  cursor: pointer;
  font-size: 18px;
  border-radius: 50%;
}
.enterTodo {
  display: flex;
  flex-direction: column;
  gap: 20px;
  width: 600px;
}
.enterTodo input[type="text"] {
  width: 100%;
  height: 40px;
  border-radius: 25px;
  font-size: 20px;
  padding: 0 20px;
  text-transform: capitalize;
}
.enterTodo input[type="text"]:focus {
  outline: none;
}

.enterTodo select {
  width: 150px;
  height: 40px;
  cursor: pointer;
  background: blue;
  color: white;
  border: none;
  font-size: 18px;
  font-weight: 500;
  border-radius: 9px;
}

.enterTodo button {
  width: 90px;
  height: 35px;
  background: blue;
  color: white;
  font-size: 20px;
  font-weight: 500;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.enterTodo button:hover {
  background: green;
  transition: 0.5s;
}
.todo-wrapper {
  max-height: 400px;
  overflow-y: scroll;
}
.todos {
  height: 35px;
  width: 400px;
  position: relative;
  margin: 30px 0;
}
.todos input[type="text"] {
  font-size: 18px;
  width: 100%;
  height: 100%;
  border-radius: 25px;
  padding: 0 10px;
  border: none;
}
.todos input[type="text"]:focus {
  outline: none;
}
.todos .very-important {
  background: green;
  color: white;
}
.todos .important {
  background: blue;
  color: white;
}
.todos .crud {
  position: absolute;
  top: 50%;
  right: 3%;
  transform: translateY(-50%);
}
.crud .deleteBtn {
  margin: 0 5px;
  cursor: pointer;
  /* border: none; */
  background: red;
  color: white;
}

.crud .completeBtn,
.editBtn {
  cursor: pointer;
}
.editBtn {
  background: greenyellow;
  cursor: pointer;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}
@media (max-width: 640px) {
  .name {
    width: 350px;
  }
  .enterTodo {
    width: 400px;
  }
}
@media (max-width: 430px) {
  .name {
    width: 300px;
  }
  .enterTodo {
    width: 320px;
  }
  .todos {
    width: 100%;
  }
  .todos input[type="text"] {
    font-size: 13px;
    width: 100%;
  }
}
</style>
