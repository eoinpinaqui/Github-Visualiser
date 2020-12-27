<template>
  <div>
    <Unknown v-if="!display" :search="toSearch" category="Repositories"></Unknown>
    <v-container v-else>
      <v-card
          elevation="2"
          style="padding: 3rem"
      >
        <v-card-title
            class="justify-center"
            style="font-size: 3rem; padding-top: 1em"
        >{{ orgName }}
        </v-card-title>
        <v-card-text
            align="center"
        >{{ orgDescription }}
        </v-card-text>
        <v-row
            align="center"
            justify="center"
        >
          <v-col>
            <v-img
                max-width="300px"
                :src=avatarURL
                style="margin: 0 auto; border-radius: 50%"
            ></v-img>
          </v-col>
          <v-col>
            <div
                style="margin: 1em"
            >
              <h1>{{ name }}</h1>
              <p>
                <v-icon
                    medium
                >mdi-calendar
                </v-icon>
                Operating since {{ created }}
              </p>
              <p>
                <v-icon
                    medium
                >mdi-google-maps
                </v-icon>
                Based in {{ location }}
              </p>
              <p>
                <v-icon
                    medium
                >mdi-source-repository
                </v-icon>
                {{ numPublicRepos }} repositories
              </p>
            </div>
          </v-col>
        </v-row>
        <v-row
        >
          <v-col>
            <p>{{orgData}}</p>
          </v-col>

        </v-row>
      </v-card>
    </v-container>
  </div>
</template>

<script>
import Unknown from "./Unknown";
import axios from "axios";

export default {
  name: "Repo",

  components: {
    Unknown
  },

  data: () => ({
    display: true,

    /* Extracted Organisation Information */
    orgName: "Org Name",
    orgDescription: "Org Description",
    orgData: "",
    avatarURL: "",
    name: "",
    numPublicRepos: 0,
    created: "",
    location: "",
  }),

  props: {
    toSearch: String,
    token: String,
  },

  created() {
    this.search();
  },

  watch: {
    toSearch: function () {
      this.display = true;
      this.search();
    },
    display: function () {
      if (!this.display) {
        console.log("Not found")
      }
    }
  },

  methods: {
    search() {
      this.getOrgInfo();
    },

    getOrgInfo() {
      let baseURL = "https://api.github.com";
      let org = this.toSearch;
      const urlToQuery = baseURL + "/orgs/" + org;
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            this.orgData = response.data;
            this.orgName = this.orgData.login;
            this.orgDescription = this.orgData.description;
            this.avatarURL = this.orgData.avatar_url;
            this.name = this.orgData.name;
            this.numPublicRepos = this.orgData.public_repos;
            this.formatDate(this.orgData.created_at);
            this.location = this.orgData.location;
          })
          .catch(error => {
            this.display = false;
            alert(error);
          })
    },

    formatDate(date) {
      date = date.substring(0, 10);
      let year = date.substring(0, 4);
      let month = date.substring(5, 7);

      let dateToShow = "";

      switch(month) {
        case "01":
          dateToShow += "January";
          break;
        case "02":
          dateToShow += "February";
          break;
        case "03":
          dateToShow += "March";
          break;
        case "04":
          dateToShow += "April";
          break;
        case "05":
          dateToShow += "May";
          break;
        case "06":
          dateToShow += "June";
          break;
        case "07":
          dateToShow += "July";
          break;
        case "08":
          dateToShow += "August";
          break;
        case "09":
          dateToShow += "September";
          break;
        case "10":
          dateToShow += "October";
          break;
        case "11":
          dateToShow += "November";
          break;
        case "12":
          dateToShow += "December";
          break;
        default:
          dateToShow = "";
      }

      this.created = dateToShow + " " + year;
    },

    getOrgMembers() {
      let baseURL = "https://api.github.com";
      let org = this.toSearch;
      const urlToQuery = baseURL + "/orgs/" + org + "/members";
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            console.log(response.data);
          })
          .catch(error => {
            this.display = false;
            alert(error);
          })
    },


    searchUser(user) {
      this.$emit("searchUser", user);
    },
  },
}
</script>