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
              <p v-if="location != ''">
                <v-icon
                    medium
                >mdi-account-group
                </v-icon>
                 {{ members.length }} members
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
        <v-container style="margin-top: 1em">
          <v-row
          >
            <h3>Top 100 repositories by number of commits:</h3>
            <v-col>
              <column-chart :data="repoCommits"></column-chart>
            </v-col>

          </v-row>
        </v-container>

        <v-row
        >

          <v-col>
            <p>{{orgData}}</p>
          </v-col>
          <v-col>
            <p>{{members}}</p>
          </v-col>
          <v-col>
            <p>{{repoCommits}}</p>
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
    members: [],
    repos: [],
    repoCommits: []
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
      this.repoCommits = [];
      this.getOrgInfo();
      this.getOrgMembers(1);
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
            this.getRepos(this.orgData.repos_url, 1);
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

    getOrgMembers(page) {
      let baseURL = "https://api.github.com";
      let org = this.toSearch;
      const urlToQuery = baseURL + "/orgs/" + org + "/members?per_page=100&page=" + page;
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            for(let i = 0; i < response.data.length; i++) {
              this.members.push(response.data[i]);
            }
            if(response.data.length === 100) {
              this.getOrgMembers(page + 1);
            }
          })
          .catch(error => {
            this.display = false;
            alert(error);
          })
    },

    getRepos(reposURL, page) {
      let urlToQuery = reposURL + "?per_page=100&page=" + page;
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {

            for(let i = 0; i < response.data.length; i++) {
              this.repos.push(response.data[i]);
            }
            if(response.data.length === 100) {
              this.getRepos(reposURL, page + 1);
            } else {
              for(let i = 0; i < this.repos.length; i++) {
                this.getContriubtors(this.repos[i].name, this.repos[i].contributors_url);
              }
            }
          })
          .catch(error => {
            this.display = false;
            alert(error);
          })
    },

    getContriubtors(repoName, urlToQuery) {
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
          })
          .then(response => {
            let numCommits = 0;
            for(let i = 0; i < response.data.length; i++) {
              numCommits += response.data[i].contributions;
            }

            if(this.repoCommits.length < 100 || (this.repoCommits[this.repoCommits.length - 1][1] < numCommits)) {
              this.repoCommits.push([repoName, numCommits]);

              this.repoCommits.sort((a, b) => (a[1] > b[1]) ? -1 : (b[1] > a[1]) ? 1 : 0);
              this.repoCommits = this.repoCommits.slice(0, 100);
            }

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