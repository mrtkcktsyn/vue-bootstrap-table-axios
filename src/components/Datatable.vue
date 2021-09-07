<template>
  <div class="container pt-2 pb-5">
    <b-row>
      <b-col>
        <b-table
          id="my-table"
          responsive="sm"
          hover
          :items="items"
          :per-page="perPage"
          :current-page="currentPage"
          sort-icon-left
          :sort-desc.sync="sortDesc"
          :sort-by.sync="sortBy"
          no-provider-sorting
          :fields="fields"
        >
          <template #cell(id)="items">
            {{ items.item.id }}
          </template>

          <template #cell(title)="items">
            {{ items.item.title }}
          </template>

          <template #cell(name)="items">
            {{ items.item.name }}
          </template>

          <template #cell(status)="items" sortable="true" sortItems>
            {{ isStatus(items.item.completed) }}
          </template>

          <template #cell(actions)="items">
            <b-button
              v-b-modal.modal-center
              variant="primary"
              @click="getItem(items.item.id - 1, items.item.completed)"
              >Edit</b-button
            >
            <b-button variant="danger" @click="deleteItem(items.item.id)"
              >Delete</b-button
            >
          </template>
        </b-table>

        <b-pagination
          pills
          v-model="currentPage"
          :total-rows="rows"
          :per-page="perPage"
          limit="11"
          prev-text="Prev"
          next-text="Next"
          aria-controls="my-table"
        ></b-pagination>
        
      </b-col>
    </b-row>

    <b-modal id="modal-center" centered title="Edit Todo">
      <b-form-input
        v-model="todoTitle"
        placeholder="Enter new title here..."
      ></b-form-input>

      <b-form-checkbox
        id="checkbox-1"
        v-model="todoStatus"
        name="checkbox-1"
        checked="true"
        unchecked-value="false"
      >
        Completed
      </b-form-checkbox>

      <template #modal-footer>
        <b-button variant="success" @click="save()">Save</b-button>
        <b-button variant="danger" @click="this.$bvModal.hide('modal-center')">Exit</b-button>
      </template>
    </b-modal>
  </div>
</template>

<script>
import axios from "axios";
import config from "@/config";

export default {
  name: "App",

  data: () => ({
    items: [],
    todos: [],
    users: [],
    todoTitle: "",
    todoStatus: {
      default: null,
      type: Boolean,
    },
    todoId: null,
    showModal: false,
    perPage: 10,
    currentPage: 1,
    sortBy: "status",
    sortDesc: false,
    fields: [
      { key: "id", label: "#" },
      { key: "title", label: "Title" },
      { key: "name", label: "Assignee" },
      { key: "status", label: "Status", sortable: true },
      { key: "actions", label: "Actions" },
    ],
  }),

  computed: {
    rows() {
      return this.items.length;
    },
  },

  methods: {
    async getTodos() {
      let response = await axios.get(`${config.apiUrl}/todos`);
      this.todos = response.data;
    },

    async getUsers() {
      await axios.get(`${config.apiUrl}/users`).then((res) => {
        this.users = res.data;
      });

      setTimeout(() => {
        this.getName(this.todos, this.users);
      }, 100);
    },

    getName(todos, users) {
      var res = [];
      res = todos.map((obj) => {
        var index = users.findIndex((el) => el["id"] === obj["userId"]);
        var { name } = index !== -1 ? users[index] : {};
        return {
          ...obj,
          name,
        };
      });
      return (this.items = res);
    },

    async deleteItem(itemId) {
      await axios.delete(`${config.apiUrl}/todos/${itemId}`).then(() => {
        let i = this.items.map((data) => data.id).indexOf(itemId);
        this.items.splice(i, 1);
      });
    },

    async save() {
      await axios
        .patch(`${config.apiUrl}/todos/${this.todoId}`, {
          title: this.todoTitle,
          completed: JSON.parse(this.todoStatus),
          id: this.todoId,
        })
        .then((res) => {
          let i = this.items.map((data) => data.id).indexOf(this.todoId);
          if (this.todoTitle != "" && this.todoTitle != null) {
            this.items[i].title = res.data.title;
          }
          this.items[i].completed = res.data.completed;
        });

      this.todoTitle = "";

      this.$bvModal.hide("modal-center");
    },

    getItem(id, completed) {
      this.todoId = id + 1;
      this.todoStatus = completed;
    },

    isStatus(item) {
      /* item === true ? (item = 1) : (item = 0); */
      return item === 0 ? "In Progress" : "Done";
    },
  },

  mounted() {
    this.getTodos();
    this.getUsers();
  },
};
</script>

<style lang="scss">
</style>