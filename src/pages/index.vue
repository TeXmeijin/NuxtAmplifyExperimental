<template>
  <amplify-authenticator>
    <h1>TodoApp</h1>
    <b-field v-model="name" label="Name"></b-field>
    <b-field v-model="description" label="Description"></b-field>
    <v-btn @click="createTodo">Create</v-btn>
    <ul>
      <li v-for="todo in todos" :key="todo.id">
        {{ todo.name }} : {{ todo.description }}
      </li>
    </ul>
    <amplify-sign-out></amplify-sign-out>
  </amplify-authenticator>
</template>

<script lang="ts">
import API, { graphqlOperation, GraphQLResult } from '@aws-amplify/api'
import Vue from 'vue'
import { createTodo } from '~/src/graphql/mutations'
import { listTodos } from '~/src/graphql/queries'
import { onCreateTodo } from '~/src/graphql/subscriptions'

type Item = { id: number; name: string; description: string }

export default Vue.extend({
  data() {
    return {
      name: '',
      description: '',
      todos: [] as Item[],
    }
  },
  async created() {
    await this.getTodos()
  },
  methods: {
    async createTodo() {
      const { name, description } = this
      if (!name || !description) return false
      const todo = { name, description }
      await API.graphql({
        query: createTodo,
        variables: { input: todo },
      })
      this.name = ''
      this.description = ''
    },
    async getTodos() {
      const todos = (await API.graphql(
        graphqlOperation(listTodos)
      )) as GraphQLResult<Item>
      this.todos = todos.data.listTodos.items
    },
    subscribe() {
      API.graphql({ query: onCreateTodo }).subscribe({
        next: (eventData: any) => {
          const todo = eventData.value.data.onCreateTodo
          if (this.todos.some((item) => item.name === todo.name)) return // remove duplications
          this.todos = [...this.todos, todo]
        },
      })
    },
  },
})
</script>
