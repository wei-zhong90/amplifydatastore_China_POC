<template>
  <div id="app">
    <h1>Todo App</h1>
    <input type="text" v-model="name" placeholder="Todo name">
    <input type="text" v-model="description" placeholder="Todo description">
    <button v-on:click="createTodo">Create Todo</button>
    <button v-on:click="deleteTodo">Delete Todo</button>
    <div v-for="item in todos" :key="item.id">
      <h3>{{ item.name }}</h3>
      <p>{{ item.description }}</p>
    </div>
    </div>

</template>

<script>
import { DataStore } from 'aws-amplify';
import { Todo } from './models';
// import { createTodo } from './graphql/mutations';
// import { listTodos } from './graphql/queries';
// import { onCreateTodo } from './graphql/subscriptions';

export default {
  name: 'App',
  async created() {
    this.subscribe();
    this.getTodos();
  },
  data() {
    return {
      name: '',
      description: '',
      todos: []
    }
  },
  methods: {
    async deleteTodo() {
      try {
        await DataStore.clear();
      } catch(error) {
        console.log("The error appears in the delete action");
        console.error(error);
      }
      this.todos = [];
    },
    async createTodo() {
      const { name, description } = this;
      if (!name || !description) return;
      const todo = { name, description };
      this.todos = [...this.todos, todo];
      // await API.graphql({
      //   query: createTodo,
      //   variables: {input: todo},
      // });
      await DataStore.save(
        new Todo(
          {
            name: todo.name,
            description: todo.description
          }
        )
      );
      this.name = '';
      this.description = '';
    },
    async getTodos() {
      const todos = await DataStore.query(Todo);
      // console.log(todos);
      this.todos = todos;
    },
    subscribe() {
      try {
        DataStore.observe(Todo)
        .subscribe(
          (eventData) => {
            let todo = eventData.element;
            // console.log("to do list is: ", todo)
            if (this.todos.some(item => item.name === todo.name)) return; // remove duplications
            this.todos = [...this.todos, todo];
          }
        );
      } catch (error) {
        console.log("the error is: ");
        console.error(error);
    }
    } 
  }
}
</script>