<template>
  <div class="text-center my-5">
    <div v-if="fetchStatus === API_STATE_ENUM.PENDING" class="my-5">
      <ProgrssBar />
    </div>
    <v-form
      v-model="valid"
      ref="form"
      lazy-validation
      class="px-5"
      @submit.prevent="addUser"
    >
      <v-text-field
        v-model="first_name"
        :rules="nameRules"
        label="First name"
        required
      ></v-text-field>

      <v-text-field
        v-model="last_name"
        label="Last name"
        :rules="nameRules"
        required
      ></v-text-field>
      <v-text-field
        v-model="email"
        :rules="emailRules"
        label="Email"
        required
      ></v-text-field>
      <v-file-input
        accept="image/png, image/jpeg, image/bmp"
        placeholder="Pick an avatar"
        prepend-icon="mdi-camera"
        label="Avatar"
        @change="uploadImage"
        v-model="image"
      ></v-file-input>
      <v-btn
        color="orange lighten-2"
        :disabled="!valid"
        class="my-4"
        text
        @click="addUser"
      >
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
  data() {
    return {
      valid: true,
      email: "",
      last_name: "",
      first_name: "",
      id: "",
      image: null,
      base64Image: null,
      API_STATE_ENUM,
      fetchStatus: API_STATE_ENUM.IDLE,
      nameRules: [v => !!v || "Name is required"],
      emailRules: [
        v => !!v || "E-mail is required",
        v => /.+@.+\..+/.test(v) || "E-mail must be valid"
      ]
    };
  },
  props: {
    action: String,
    user: Object
  },
  methods: {
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
    },
    async addUser() {
      if (this.$refs.form.validate()) {
        this.fetchStatus = API_STATE_ENUM.PENDING;
        const axios = require("axios");
        if (this.action === "add") {
          const req = await axios.post(`https://reqres.in/api/users`, {
            email: this.email,
            firstname: this.first_name,
            lastname: this.last_name,
            userImage: this.base64Image
              ? this.base64Image
              : "https://reqres.in/img/faces/7-image.jpg"
          });
          this.fetchStatus = API_STATE_ENUM.RESOLVED;
          this.$emit("close");
          this.$emit("addUser", req.data);
        }
        if (this.action === "update") {
          const req = await axios.put(
            `https://reqres.in/api/users/${this.user.id}`,
            {
              email: this.email,
              firstname: this.first_name,
              lastname: this.last_name,
              userImage: this.base64Image
                ? this.base64Image
                : this.user.userImage
            }
          );
          this.fetchStatus = API_STATE_ENUM.RESOLVED;
          this.$emit("close");
          this.$emit("updateUser", req.data);
        }
      }
      // this.$refs.form.reset();
    }
  },
  components: {
    ProgrssBar
  }
};
</script>
