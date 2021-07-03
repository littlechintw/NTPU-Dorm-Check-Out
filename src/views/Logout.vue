<template>
  <v-card elevation="0">
    <h1>Logout...</h1>
    <v-overlay>
      <v-progress-circular indeterminate size="64"> </v-progress-circular>
    </v-overlay>
  </v-card>
</template>

<script>
const axios = require("axios");
var config = require("../../config.json");

export default {
  name: "Home",
  data() {
    return {};
  },
  components: {},
  methods: {
    logout() {
      axios.post(config.apiurl + "/account/logout", {
        stu_id: this.stuid,
        pwd: this.pwd,
      });
    },
  },
  mounted: function () {
    if (!this.$cookie.get("session") || !this.$cookie.get("id")) {
      this.$router.push("/");
    } else {
      this.$cookie.delete("session");
      this.$cookie.delete("id");
      this.logout();
      this.$router.push("/");
      location.reload();
    }
  },
};
</script>
