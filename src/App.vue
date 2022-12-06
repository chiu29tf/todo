<script lang="ts">
const STORAGE_KEY = "my-todomvc";

const filters = {
  all: (todos: any) => todos, active: (todos: any) => todos.filter((todo) => !todo.completed), completed: (todos: any) => todos.filter((todo) => todo.completed)
};

export default {
    data: () => ({
        todos: JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
        , editedTodo: null
        , visibility: 'all'
    })
    , watch: {
        todos: {
            handler(todos) {
                localStorage.getItem(STORAGE_KEY, JSON.stringify(todos))
            }
            , deep: true
        }
    }
    , mounted() {
        window.addEventListener('hashchange', this.onHashChange)
        this.onHashChange()
    }
    , computed: {
        filteredTodos() {
            return filters[this.visibility](this.todos)
        }
        , remaining() {
            return filters.active(this.todos).length
        }
    }
    , methods: {
        toggleAll(e) {
            this.todos.forEach((todo) => (todo.completed = e.target.checked))
        }
        , addTodo(e) {
            const value = e.target.value.trim()
            if (!value) { return }
            this.todos.push({
                id: Date.now()
                , title: value
                , completed: false
            })
            e.target.value = ''
        }
        , removeTodo (todo) {
            this.todos.splice(this.todos.indexOf(todo), 1)
        }
        , editTodo (todo) {
            this.beforeEditCache = todo.title
            this.editedTodo = todo
        }
        , doneEdit (todo) {
            if (!this.editedTodo) { return }
            this.editedTodo = null
            todo.title = todo.title.trim()
            if (!todo.title) { this.removeTodo(todo) }
        }
        , cancelEdit(todo) {
            this.editedTodo = null
            todo.title = this.beforeEditCache
        }
        , removeCompleted () {
            this.todos = filters.active(this.todos)
        }
        , onHashChange () {
            var visibility = window.location.hash.replace(/#\/?/, '')
            if (filters[visibility]) {
                this.visibility = visibility
            } else {
                window.location.hash = ''
                this.visibility = 'all'
            }
        }
    }
}
</script>

<template>
    <section class="todoapp">
        <header class="header">
            <h1>todos</h1>
            <input type="text" class="new-todo" autofocus placeholder="What need to be done?" @keyup.enter="addTodo">
        </header>
        <section class="main" v-show="todos.length">
            <input type="checkbox" id="toggle-all" class="toggle-all" :checked="remaining === 0" @change="toggleAll">
            <label for="toggle-all">Mark all as complete</label>
            <ul class="todo-list">
                <li v-for="todo in filteredTodos" class="todo" :key="todo.id" :class="{ completed: todo.completed, editing: todo === editedTodo}">
                    <div class="view">
                        <input type="checkbox" class="toggle" model="todo.completed">
                        <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
                        <button class="destroy" @click="removeTodo(todo)"></button>
                    </div>
                    <input type="text" v-if="todo === editedTodo" class="edit" v-model="todo.title" @vnode-mounted="({ el }) => el.focus()" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.escape="cancelEdit(todo)" >
                </li>
            </ul>
        </section>
        <footer class="footer" v-show="todos.length">
            <span class="todo-count">
                <strong>{{ remaining }}</strong>
                <span>{{ remaining === 1 ? ' item' : ' items' }} left</span>
            </span>
            <ul class="filters">
                <li>
                    <a href="#/all" :class="{ selected: visibility === 'all' }">All</a>
                </li>
                <li>
                    <a href="#/active" :class="{ selected: visibility === 'active' }">Active</a>
                </li>
                <li>
                    <a href="#/completed" :class="{ selected: visibility === 'completed' }">Completed</a>
                </li>
            </ul>
            <button class="clear-completed" @click="removeCompleted" v-show="todos.length > remaining">Clear completed</button>
        </footer>
    </section>
</template>

<style>
@import "https://unpkg.com/todomvc-app-css@2.4.1/index.css";
</style>