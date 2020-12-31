<template>
  <div>
    <Unknown v-if="!display" :search="toSearch" category="Organisations"></Unknown>
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
            <v-col>
              <h3>Top {{ repoCommits.length}} repositories by number of commits:</h3>
              <column-chart :data="repoCommits"></column-chart>
            </v-col>
            <v-col>
              <h3>Top {{ shortUserCommits.length}} members by number of commits in all repositories:</h3>
              <column-chart :data="shortUserCommits"></column-chart>
            </v-col>

          </v-row>
        </v-container>
        <v-row>
          <v-col>
            <v-card
                elevation="2"
                style="padding: 1rem; margin: 0.5rem"
            >
              <v-card-title
                  class="justify-center"
                  style="font-size: 2rem; padding-top: 1em"
              >Organisation Heroes
              </v-card-title>
              <v-card-text
                  align="center"
              >"Hero Rating" is a metric (devised by Eoin Pinaqui for the CSU33012 Software Engineering module) that
                can be used to rank organization members based on their involvement in the organizations repositories.
              </v-card-text>
              <v-card v-on:click="searchUser(user.login)" v-for="user in heroRatings.slice(0, 4)" :key="user"
                      elevation="2"
                      class="justify-center"
                      style="padding: 1rem; margin: 2em auto; text-align: center; max-width: 500px">
                <div style="display: flex; flex-direction: row; align-items: center">
                  <v-icon>mdi-cards-heart</v-icon>
                  <img style="margin: 0 1em; width: 10em; border-radius: 50%" :src=user.image />
                  <p style="margin: 0; font-size: 2em"><strong>{{ user.login }}</strong></p>
                </div>
              </v-card>
            </v-card>
          </v-col>
        </v-row>
        <v-row style="margin: 1em">
          <v-col>
            <h3 style="text-align: center">Click below to learn more about an organisation member:</h3>
            <div style="display: flex; flex-direction: row; align-items: center; flex-wrap: wrap">
              <v-card v-on:click="searchUser(user.login)" v-for="user in members" :key="user"
                      elevation="2"
                      style="padding: 1rem; margin: 0.5rem; width: 19%; text-align: center">
                <div style="display: flex; flex-direction: row; align-items: center">
                  <img style="margin: 0 1em;width: 3rem; border-radius: 50%" :src=user.avatar_url />
                  <p style="margin: 0"><strong>{{ user.login }}</strong></p>
                </div>
              </v-card>
            </div>
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
    orgName: "",
    orgDescription: "",
    orgData: "",
    avatarURL: "",
    name: "",
    numPublicRepos: 0,
    created: "",
    location: "",
    members: [],
    repos: [],
    repoCommits: [],
    userRepos: [],
    userCommits: [],
    shortUserRepos: [],
    shortUserCommits: [],
    heroRatings: [],
    shortHeroRatings: [],
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
      this.orgName = "";
      this.orgDescription = "";
      this.orgData = "";
      this.avatarURL = "";
      this.name = "";
      this.numPublicRepos = 0;
      this.created = "";
      this.location = "";
      this.members = [];
      this.repos = [];
      this.repoCommits = [];
      this.userCommits = [];
      this.userRepos = [];
      this.shortUserRepos = [];
      this.shortUserCommits = [];
      this.heroRatings = [];
      this.shortHeroRatings = [];
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
            this.display = true;
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
            console.log(error);
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
            console.log(error);
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
            console.log(error);
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

              let found = false;
              for(let j = 0; j < this.userCommits.length; j++) {
                if(this.userCommits[j][0] === response.data[i].login) {
                  this.userCommits[j][1] += response.data[i].contributions;
                  found = true;
                }
              }
              if(!found) {
                this.userCommits.push([response.data[i].login, response.data[i].contributions]);
              }

              found = false;
              for(let j = 0; j < this.userRepos.length; j++) {
                if(this.userRepos[j][0] === response.data[i].login) {
                  this.userRepos[j][1] += 1;
                  found = true;
                }
              }
              if(!found) {
                this.userRepos.push([response.data[i].login, 1, response.data[i].avatar_url]);
              }
            }

            this.userCommits.sort((a, b) => (a[1] > b[1]) ? -1 : (b[1] > a[1]) ? 1 : 0);
            this.shortUserCommits = this.userCommits.slice(0, 50);

            this.userRepos.sort((a, b) => (a[1] > b[1]) ? -1 : (b[1] > a[1]) ? 1 : 0);
            this.shortUserRepos = this.userRepos.slice(0, 50);

            if(this.repoCommits.length < 50 || (this.repoCommits[this.repoCommits.length - 1][1] < numCommits)) {
              this.repoCommits.push([repoName, numCommits]);

              this.repoCommits.sort((a, b) => (a[1] > b[1]) ? -1 : (b[1] > a[1]) ? 1 : 0);
              this.repoCommits = this.repoCommits.slice(0, 50);
            }
            this.calculateHeroRating();
          })
          .catch(error => {
            this.display = false;
            console.log(error);
          })
    },

    calculateHeroRating() {
      this.heroRatings = [];
      for(let i = 0; i < this.userCommits.length; i++) {
        for(let j = 0; j < this.userRepos.length; j++) {
          if(this.userRepos[j][0] === this.userCommits[i][0]) {
            let login = this.userRepos[j][0];
            let rating = j + i;
            let image = this.userRepos[j][2];
            let score = this.userCommits[i][1] + " " + this.userRepos[j][1];
            this.heroRatings.push({login, rating, image, score});
          }
        }
      }
      this.heroRatings.sort((a, b) => (a.rating > b.rating) ? 1 : (b.rating > a.rating) ? -1 : 0);
      this.shortHeroRatings = this.shortHeroRatings.slice(0, 3);
    },

    searchUser(user) {
      this.$emit("searchUser", user);
    },
  },
}
</script>