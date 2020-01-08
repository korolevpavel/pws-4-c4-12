<template>
  <div class="container">
    <div class="col-sm-10">
      <h1>Задачи: сделано {{ done }} - ожидают {{ doing }}</h1>
      <confirmation :message="confirmationMessage"
        v-model="showConfirmation"
        v-if="showConfirmation">
      </confirmation>
      <div class="header-btn"><button
        type="button"
        id="task-add"
        class="btn btn-success bt-sm align-left d-block"
        v-b-modal.todo-modal>
        Добавить задачу</button>
        <button
        type="button"
        id="clear-ls"
        class="btn btn-danger bt-sm align-left d-block"
        @click="clearLocalStorage">
        Очистить хранилище</button>
        </div>
      <table class="table table-dark table-stripped table-hover">
        <thead class="thead-light">
          <tr>
            <th>Uid</th>
            <th>Описание</th>
            <th>Выполнена?</th>
            <th></th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(todo, index) in todos" :key="index">
            <td class="todo-uid">{{ todo.uid }}</td>
            <td>{{ todo.description }}</td>
            <td>
              <span v-if="todo.is_completed">Выполнено</span>
              <span v-else>Не выполнено</span>
            </td>
            <td>
              <div class="btn-group" role="group">
                <button type="button"
                      class="btn btn-secondary btn-sm"
                      v-b-modal.todo-update-modal
                      @click="updateTodo(todo)"
                      >
                  Обновить
              </button>
                &nbsp;
                <button type="button"
                    class="btn btn-danger btn-sm"
                    @click="deleteTodo(todo)">X</button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <b-modal ref="addTodoModal" id="todo-modal" title="Добавить задачу" hide-footer>
      <b-form @submit="onSubmit" @reset="onReset" class="w-100">
        <b-form-group
          id="from-description-group"
          label="Описание:"
          label-for="form-description-input"
        >
          <b-form-input
            id="form-description-input"
            type="text"
            v-model="addTodoForm.description"
            required
            placeholder="Завести задачу"
          ></b-form-input>
        </b-form-group>
        <b-form-group id="form-complete-group">
          <b-form-checkbox-group v-model="addTodoForm.is_completed" id="form-checks">
            <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
          </b-form-checkbox-group>
        </b-form-group>
        <b-button type="submit" variant="primary">Добавить</b-button>
        <b-button type="reset" variant="danger">Сброс</b-button>
      </b-form>
    </b-modal>

    <b-modal ref="updateTodoModal"
            id="todo-update-modal"
            title="Update"
            hide-footer>
      <b-form @submit="onUpdateSubmit" @reset="onUpdateReset" class="w-100">
      <b-form-group id="form-update-description-group"
                    label="Описание:"
                    label-for="form-update-description-input">
        <b-form-input id="form-update-description-input"
                      type="text"
                      v-model="updateTodoForm.description"
                      required>
        </b-form-input>
      </b-form-group>
      <b-form-group id="form-update-complete-group">
        <b-form-checkbox-group v-model="updateTodoForm.is_completed" id="form-update-checks">
          <b-form-checkbox value="true">Задача выполнена?</b-form-checkbox>
        </b-form-checkbox-group>
      </b-form-group>
      <b-button-group>
        <b-button type="submit" variant="primary">Обновить</b-button>
        <b-button type="reset" variant="danger">Сброс</b-button>
      </b-button-group>
      </b-form>
    </b-modal>

  </div>
</template>

<style>
.header-btn {
  display: flex;
  margin-top: 20px;
  margin-bottom: 20px;
  justify-content: space-between;
}

h1,
td {
  text-align: left;
}

.todo-uid {
  text-align: right;
}
</style>

<script>
import Confirmation from './Confirmation.vue';

export default {
  name: 'Todo',
  data() {
    return {
      done: 0,
      doing: 0,
      todos: [],
      addTodoForm: {
        description: '',
        is_completed: [],
      },
      updateTodoForm: {
        uid: 0,
        description: '',
        is_completed: [],
      },
      confirmationMessage: '',
      showConfirmation: false,
    };
  },
  methods: {
    saveToLocalStorage() {
      localStorage.setItem('todos', JSON.stringify(this.todos));
    },
    clearLocalStorage() {
      localStorage.removeItem('todos');
      this.todos = [];
      this.updateStats();
    },
    updateStats() {
      let isDone = 0;
      let isDoing = 0;
      if (this.todos.length > 0) {
        const todos = Object.values(this.todos);
        for (let i = 0; i < todos.length; i += 1) {
          if (todos[i].is_completed !== undefined && todos[i].is_completed[0] === 'true') {
            isDone += 1;
          } else {
            isDoing += 1;
          }
        }
      }
      this.done = isDone;
      this.doing = isDoing;
    },
    getTodos() {
      if (localStorage.getItem('todos')) {
        try {
          this.todos = JSON.parse(localStorage.getItem('todos'));
          this.updateStats();
        } catch (error) {
          localStorage.removeItem('todos');
        }
      }
    },
    resetForm() {
      this.addTodoForm.description = '';
      this.addTodoForm.is_completed = [];
      this.updateTodoForm.description = '';
      this.updateTodoForm.is_completed = [];
    },
    onSubmit(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();

      const { description } = this.addTodoForm;
      let uidNew = 0;
      if (this.todos.length > 0) {
        const todos = Object.values(this.todos);
        uidNew = todos[0].uid;
        for (let i = 0; i < todos.length; i += 1) {
          if (uidNew < todos[i].uid) {
            uidNew = todos[i].uid;
          }
        }
      }
      uidNew += 1;
      if (typeof this.addTodoForm.is_completed[0] === 'undefined') {
        this.addTodoForm.is_completed = false;
      } else if (typeof this.addTodoForm.is_completed[0] === 'string') {
        if (this.addTodoForm.is_completed[0] === 'true') {
          this.addTodoForm.is_completed = true;
        } else {
          this.addTodoForm.is_completed = false;
        }
      }

      const { isCompleted } = this.addTodoForm;
      const uid = uidNew;
      this.todos.push({
        uid,
        description,
        isCompleted,
      });

      this.confirmationMessage = `Задача "${description}" добавлена`;
      this.showConfirmation = true;
      this.updateStats();
      this.saveToLocalStorage();
      this.resetForm();
    },
    onReset(event) {
      event.preventDefault();
      this.$refs.addTodoModal.hide();
      this.resetForm();
    },
    updateTodo(todo) {
      this.updateTodoForm = todo;
    },
    deleteTodo(todo) {
      this.confirmationMessage = `Задача №${todo.uid} удалена из списка`;
      this.showConfirmation = true;
      const indexDelete = this.todos.findIndex(item => item.uid === todo.uid);
      this.todos.splice(indexDelete, 1);
      this.saveToLocalStorage();
      this.updateStats();
    },
    onUpdateSubmit(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      this.confirmationMessage = `Задача №${this.updateTodoForm.uid} обновлена`;
      this.showConfirmation = true;
      this.saveToLocalStorage();
      this.updateStats();
    },
    onUpdateReset(event) {
      event.preventDefault();
      this.$refs.updateTodoModal.hide();
      this.resetForm();
    },
  },

  created() {
    this.getTodos();
  },
  components: {
    confirmation: Confirmation,
  },
};
</script>
