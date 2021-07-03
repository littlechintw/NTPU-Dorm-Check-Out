<template>
  <v-card elevation="0">
    <v-container fluid>
      <v-row>
        <v-col cols="12">
          <v-row align="center" justify="center" length><br /></v-row>
          <v-row v-show="!loginStatus" align="center" justify="center" length>
            <about />
          </v-row>

          <v-row v-show="loginStatus" align="center" justify="center" length>
            <h1>{{ userData.building }}</h1>
          </v-row>
          <v-row v-show="loginStatus" align="center" justify="center" length>
            <br />
          </v-row>
          <v-row
            v-show="userData.reserve"
            align="center"
            justify="center"
            length
          >
            <v-card>
              <v-tabs v-model="tab" background-color="primary" dark>
                <v-tab
                  v-for="item in tabItems"
                  :key="item.tab"
                  @click="changeStatusData(item.tab)"
                >
                  {{ item.tab }}
                </v-tab>
              </v-tabs>

              <v-tabs-items v-model="tab">
                <v-tab-item v-for="item in tabItems" :key="item.tab">
                  <!-- - -->
                  <v-card v-show="userData.reserve" elevation="0">
                    <v-data-table
                      :headers="headers"
                      :items="statusData"
                      items-per-page="50"
                    >
                      <template v-slot:item.date="{ item }">
                        <h3 style="color: #2b7a78">
                          {{ item.date }}
                        </h3>
                      </template>

                      <template v-slot:item.time="{ item }">
                        <h3 style="color: #2b7a78">
                          {{ item.time }}
                        </h3>
                      </template>

                      <template v-slot:item.currentPeople="{ item }">
                        <v-chip
                          :color="
                            chipGetColor(item.currentPeople, item.maxPeople)
                          "
                          dark
                        >
                          {{ item.currentPeople }}
                        </v-chip>
                      </template>

                      <template v-slot:item.uuid="{ item }">
                        <v-btn
                          v-show="item.serve"
                          @click="reserveBtn(item)"
                          elevation="2"
                          outlined
                          plain
                          raised
                          small
                          >預約 / Reserve</v-btn
                        >
                        <h3 v-show="!item.serve" style="color: #e76f51">⛔</h3>
                      </template>
                    </v-data-table>

                    <v-row align="center" justify="left" length>
                      <br /><br />
                    </v-row>
                  </v-card>
                  <!-- - -->
                </v-tab-item>
              </v-tabs-items>
            </v-card>
          </v-row>

          <v-row v-show="loginStatus" align="center" justify="center" length>
            <v-overlay :value="overlayLoading">
              <v-progress-circular
                indeterminate
                size="64"
              ></v-progress-circular>
            </v-overlay>

            <v-overlay :z-index="zIndex" :value="overlay">
              <v-card
                class="mx-auto text-left overflow-y-auto"
                outlined
                v-click-outside="overlayOutside"
              >
                <v-card-title> 確定要預約這個時段嗎？ </v-card-title>
                <v-card-subtitle> Please comfirm the time! </v-card-subtitle>
                <v-card-text>
                  <h3>{{ overlayData.date }}</h3>

                  <h3>{{ overlayData.time }}</h3>
                  <br />
                  <h3 style="color: #c75497">按下確認按鈕後，將無法修改</h3>
                  <h3 style="color: #c75497">
                    It will can't edit after confirm!
                  </h3>
                </v-card-text>
                <v-card-actions>
                  <v-btn
                    class="white--text"
                    color="#2a9d8f"
                    @click="reserveSend(overlayData.uuid)"
                  >
                    確定 / Comfirm
                  </v-btn>

                  <v-btn
                    class="white--text"
                    color="#e76f51"
                    @click="overlay = false"
                  >
                    取消 / Cancel
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-overlay>
          </v-row>
          <v-row align="center" justify="center" length><br /></v-row>
        </v-col>
      </v-row>
    </v-container>
  </v-card>
</template>

<script>
import About from "../components/About";
const axios = require("axios");
var config = require("../../config.json");

export default {
  name: "Home",
  data() {
    return {
      window_height: 100,
      window_width: 100,
      overlay: false,
      overlayLoading: false,
      overlayData: {
        date: "N/A",
        time: "N/A",
        currnetPeople: "N/A",
        maxPeople: "N/A",
        uuid: "N/A",
        serve: false,
      },
      loginStatus: false,
      userData: {
        uuid: "N/A",
        building: "Loading...",
        reserve: false,
      },
      tab: null,
      tabItems: [
        { tab: "7/16", content: "Tab 1 Content" },
        { tab: "7/17", content: "Tab 2 Content" },
        { tab: "7/18", content: "Tab 3 Content" },
        { tab: "7/19", content: "Tab 4 Content" },
        { tab: "7/20", content: "Tab 5 Content" },
        { tab: "7/21", content: "Tab 6 Content" },
        { tab: "7/22", content: "Tab 7 Content" },
      ],
      saveStatusData: [],
      statusData: [],
      headers: [
        { text: "日期 / Date", value: "date" },
        { text: "時間 / Time", value: "time" },
        { text: "目前人數 / Current People", value: "currentPeople" },
        { text: "總人數 / Max People", value: "maxPeople" },
        { text: "預約 / Reserve", value: "uuid" },
      ],
    };
  },
  components: {
    About,
  },
  methods: {
    reserveBtn(item) {
      this.window_height = window.innerHeight;
      this.window_width = window.innerWidth;
      this.overlay = !this.overlay;
      this.overlayData = item;
    },
    overlayOutside() {
      this.overlay = false;
    },
    updateStatus() {
      this.userData.reserve = false;
      this.statusData = [
        {
          date: "Loading...",
          time: "",
          currentPeople: 0,
          maxPeople: 0,
          uuid: "N/A",
          serve: false,
        },
      ];
      let self = this;
      axios
        .post(config.apiurl + "/status", {
          id: this.$cookie.get("id"),
          session: this.$cookie.get("session"),
        })
        .then(function (response) {
          console.log(response.data);
          self.$cookie.set("session", response.data.session, 1);
          if (response.data.code === 200) {
            if (response.data.message.admin === true) {
              self.$router.push("admin");
            }
            if (response.data.message.record === false) {
              self.saveStatusData = response.data.message.data;
              self.userData.building = response.data.message.build;
              self.userData.reserve = true;
              self.changeStatusData("7/16");
            } else {
              self.userData.building =
                "已經完成預約，請在 " +
                response.data.message.data +
                " 完成離宿工作！";
            }
          } else {
            self.$cookie.set("session", response.data.session, 1);
          }
        })
        .catch(function (error) {
          alert(error);
        });
    },
    reserveSend(eventId) {
      this.overlayLoading = true;
      this.overlay = false;
      this.userData.reserve = false;
      let self = this;
      axios
        .post(config.apiurl + "/reserve", {
          id: this.$cookie.get("id"),
          session: this.$cookie.get("session"),
          eventId: eventId,
        })
        .then(function (response) {
          self.overlayLoading = false;
          self.$cookie.set("session", response.data.session, 1);
          if (response.data.code === 200) {
            if (response.data.message.check) {
              self.userData.building =
                "已經完成預約，請在 " +
                self.overlayData.date +
                " " +
                self.overlayData.time +
                " 完成離宿工作！";
            } else {
              self.userData.reserve = true;
              self.userData.building = "預約失敗，人數已滿或停止預約";
              self.statusData = response.data.message.data;
            }
          } else {
            self.$cookie.set("session", response.data.session, 1);
          }
        })
        .catch(function (error) {
          alert(error);
        });
    },
    chipGetColor(current, max) {
      if (current >= max) return "#FF4F4F";
      else if (current / max <= 0.2) return "#4F9EFF";
      else if (current / max > 0.2 && current / max <= 0.6) return "#9EB53E";
      else if (current / max > 0.6) return "#FFAC4F";
    },
    changeStatusData(date) {
      var tmp = [];
      for (var status in this.saveStatusData) {
        if (this.saveStatusData[status]["date"] === date) {
          tmp.push(this.saveStatusData[status]);
        }
      }
      this.statusData = tmp;
    },
  },
  mounted: function () {
    if (this.$cookie.get("session") && this.$cookie.get("id")) {
      this.loginStatus = true;
      this.updateStatus();
    }
    this.window_height = window.innerHeight;
    this.window_width = window.innerWidth;
  },
};
</script>
