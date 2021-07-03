<template>
  <v-app style="background-color: #feffff">
    <div class="inapp">
      <v-app-bar app dark dense flat color="#264653">
        <v-toolbar-title>北大宿舍離宿預約</v-toolbar-title>
        <v-spacer></v-spacer>
        <v-btn :href="btn.url" elevation="2" outlined plain raised>{{
          btn.title
        }}</v-btn>
      </v-app-bar>

      <v-main>
        <v-card v-show="account">
          <v-alert type="success" dense text dismissible dense>
            已藉由 {{ account }} 登入
          </v-alert>
        </v-card>
        <div id="router" v-bind:style="{ minHeight: Height + 'px' }">
          <router-view />
        </div>
      </v-main>
    </div>

    <v-footer padless class="font-weight-medium" color="#e9c46a">
      <v-col class="text-center" cols="12" style="font-size: 14px">
        &copy; 2021 Copyright
        <a href="https://littlechintw.github.io">littlechin.tw</a>. All rights
        reserved.
      </v-col>
    </v-footer>
  </v-app>
</template>

<script>
let Base64 = require("js-base64").Base64;
export default {
  name: "App",
  components: {
    // HelloWorld,
  },
  data: () => ({
    btn: {
      title: "LOGIN",
      url: "/login",
    },
    account: "",
    login: false,
    Height: 0,
    Width: 0,
  }),
  mounted() {
    if (this.$cookie.get("id")) {
      this.btn.title = "LOGOUT";
      this.btn.url = "/logout";
      this.account = Base64.decode(this.$cookie.get("id"));
    }
    this.Height = document.documentElement.clientHeight - 90;
    this.Width = document.documentElement.clientWidth;
    window.onresize = () => {
      this.Height = document.documentElement.clientHeight - 100;
    };
  },
};
</script>

<style>
.inapp {
  width: 80%;
  margin-left: 10%;
}
#app {
  font-family: Noto Sans SC;
}
</style>