<template>
  <div class="text-center my-5 py-5">
    <!-- <div v-if="fetchStatus === API_STATE_ENUM.PENDING" class="my-5"></div> -->

    Are you sure you want to delete this user?
    <div class="justify-space-between my-5">
      <v-btn @click="cancel">No </v-btn>
      <v-btn class="error mx-4" @click="proceed">Yes </v-btn>
    </div>
  </div>
</template>
<script>
import { API_STATE_ENUM } from "/services/constant.js";
import ProgrssBar from "../Progress/index.vue";
export default {
  name: "DeleteUser",
  components: {
    ProgrssBar
  },
  data() {
    return {
      API_STATE_ENUM,
      fetchStatus: API_STATE_ENUM.IDLE
    };
  },
  methods: {
    async proceed() {
      this.fetchStatus = API_STATE_ENUM.PENDING;
      try {
        this.$emit("confrim");
        this.$emit("close");

        this.fetchStatus = API_STATE_ENUM.RESOLVED;
      } catch (error) {
        console.log(error);
      }
    },
    cancel() {
      this.$emit("hide");
    }
  }
};
</script>
