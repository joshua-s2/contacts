<template>
  <div class="container">
    <div v-if="fetchStatus === API_STATE_ENUM.PENDING" class="my-4">
      <ProgrssBar />
    </div>
    <div v-if="fetchStatus === API_STATE_ENUM.RESOLVED">
      <v-container>
        <v-btn color="orange lighten-2 my-5 ml-xl-10" @click="showModal">
          Add user
          <v-icon>
            mdi-plus
          </v-icon>
        </v-btn>
        <v-row>
          <v-col
            v-for="items in record"
            :key="items.id"
            cols="12"
            md="3"
            sm="4"
          >
            <v-card class="mx-auto my-5" max-width="344">
              <v-img :src="items.userImage" height="200px"></v-img>

              <v-card-title>
                {{ items.firstname }} {{ items.lastname }}
              </v-card-title>

              <v-card-subtitle> Email: {{ items.email }} </v-card-subtitle>

              <v-card-actions class="my-4">
                <v-btn color="white lighten-2" text @click="updateUser(items)">
                  Edit
                </v-btn>
                <v-btn color="error lighten-2" text @click="deleteModal">
                  delete
                  <v-icon left>
                    mdi-delete
                  </v-icon>
                </v-btn>
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
            <v-dialog v-model="dialog" width="500">
              <AddUserModal
                @close="dialog = false"
                @addUser="increaseUsers"
                @updateUser="update"
                :user="user"
                :action="action"
              />
            </v-dialog>
            <v-dialog v-model="confirmDelete" width="500">
              <DeleteUser
                @confrim="deleteUser(items.id)"
                @close="confirmDelete = !confirmDelete"
                @hide="confirmDelete = !confirmDelete"
              />
            </v-dialog>
          </v-col> </v-row
      ></v-container>
      <div class="text-center">
        <v-pagination
          v-model="page"
          @input="paginate"
          :length="length"
          circle
          color="orange"
        ></v-pagination>
      </div>
    </div>
  </div>
</template>
<script>
import AddUserModal from "../Modals/AddUser.vue";
import ProgrssBar from "../Progress/index.vue";
import DeleteUser from "../Modals/DeleteUser.vue";

import { API_STATE_ENUM } from "/services/constant.js";
export default {
  name: "ContactCard",

  data: () => ({
    show: false,
    record: [],
    email: "",
    dialog: false,
    confirmDelete: false,
    API_STATE_ENUM,
    action: "",
    user: {},
    page: 1,
    length: 1,
    fetchStatus: API_STATE_ENUM.IDLE,
    deleteStatus: API_STATE_ENUM.IDLE
  }),
  mounted() {
    this.getUsers(1);
  },
  methods: {
    async getUsers(currentPage) {
      try {
        this.fetchStatus = API_STATE_ENUM.PENDING;
        const axios = require("axios");
        const users = await axios.get(
          `https://reqres.in/api/users?page=${currentPage}&per_page=9`
        );
        this.record = users.data.data.map(item => ({
          email: item.email,
          firstname: item.first_name,
          lastname: item.last_name,
          id: item.id,
          userImage: item.avatar
        }));
        this.page = users.data.page;
        this.length = users.data.total_pages;
        this.fetchStatus = API_STATE_ENUM.RESOLVED;
      } catch (error) {
        console.log(error);
        this.fetchStatus = API_STATE_ENUM.REJECTED;
      }
    },
    showModal() {
      this.action = "add";
      this.dialog = true;
    },
    async deleteModal() {
      this.confirmDelete = true;
    },
    async deleteUser(id) {
      try {
        this.deleteStatus = API_STATE_ENUM.PENDING;
        const axios = require("axios");
        const url = await axios.delete(`https://reqres.in/api/users/${id}`);
        const updated = this.record.filter(items => {
          return items.id !== id;
        });
        this.record = updated;
        this.deleteStatus = API_STATE_ENUM.RESOLVED;
      } catch (error) {
        console.log(error.response);
        this.fetchStatus = API_STATE_ENUM.REJECTED;
      }
    },
    updateUser(user) {
      this.action = "update";
      this.user = user;
      this.dialog = true;
    },
    update(user) {
      const userID = this.record.findIndex(item => item.id === this.user.id);
      this.record[userID] = {
        id: userID,
        ...user
      };
      this.dialog = false;
    },
    increaseUsers(user) {
      this.record.push(user);
    },
    paginate(e) {
      this.getUsers(e);
    }
  },
  components: {
    AddUserModal,
    ProgrssBar,
    DeleteUser
  }
};
</script>
