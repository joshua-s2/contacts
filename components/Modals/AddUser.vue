<template>
  <div class="text-center my-5">
    <div v-if="fetchStatus === API_STATE_ENUM.PENDING" class="my-5">
      <ProgrssBar />
    </div>
    <v-form ref="form" class="px-5" @submit.prevent="submit">
      <v-text-field
        v-model="first_name"
        :counter="10"
        label="First name"
        required
      ></v-text-field>

      <v-text-field
        v-model="last_name"
        :counter="10"
        label="Last name"
        required
      ></v-text-field>
      <v-text-field v-model="email" label="Email" required></v-text-field>
      <v-file-input
        accept="image/png, image/jpeg, image/bmp"
        placeholder="Pick an avatar"
        prepend-icon="mdi-camera"
        label="Avatar"
        @change="uploadImage"
        v-model="image"
      ></v-file-input>
      <v-btn color="orange lighten-2" class="my-4" text @click="addUser">
        Save
      </v-btn>
    </v-form>
  </div>
</template>
<script>
import { API_STATE_ENUM } from "/services/constant.js";
import ProgrssBar from "../Progress/index.vue";
export default {
  name: "AddUserModal",
  mounted() {
    if (this.action === "update") {
      (this.last_name = this.user.lastname),
        (this.first_name = this.user.firstname),
        (this.email = this.user.email);
    }
  },
  data: () => ({
    email: "",
    last_name: "",
    first_name: "",
    image: null,
    base64Image: null,
    API_STATE_ENUM,
    fetchStatus: API_STATE_ENUM.IDLE
  }),
  props: {
    action: String,
    user: Object
  },
  methods: {
    async addUser() {
      this.fetchStatus = API_STATE_ENUM.PENDING;
      const axios = require("axios");
      if (this.action === "add") {
        const req = await axios.post(`https://reqres.in/api/users`, {
          email: this.email,
          firstname: this.first_name,
          lastname: this.last_name,
          userImage: this.base64Image
        });
        this.$emit("close");
        this.$emit("addUser", req.data);
        this.fetchStatus = API_STATE_ENUM.RESOLVED;
      } else {
        const req = await axios.put(
          `https://reqres.in/api/users/${this.user.id}`,
          {
            email: this.email,
            firstname: this.first_name,
            lastname: this.last_name,
            userImage: this.base64Image ? this.base64Image : this.user.userImage
          }
        );
        this.$emit("close");
        this.$emit("updateUser", req.data);
        this.fetchStatus = API_STATE_ENUM.RESOLVED;
      }
      this.$refs.form.reset();
    },
    async uploadImage() {
      if (this.action === "add") {
        const fileToBase64 = async file =>
          new Promise((resolve, reject) => {
            const reader = new FileReader();
            console.log(reader);
            reader.readAsDataURL(file);
            reader.onload = () => resolve(reader.result);
            reader.onerror = e => reject(e);
          });
        try {
          this.base64Image = await fileToBase64(this.image);
        } catch (error) {
          this.$sentry.captureException(error);
        }
      }
    }
  },
  components: {
    ProgrssBar
  }
};
</script>
