<template>
  <div>
    <Unknown v-if="!display" :search="toSearch" category="Users"></Unknown>
    <v-container v-else>
      <v-card
          elevation="2"
          style="padding: 3rem"
      >
        <v-card-title
            class="justify-center"
            style="font-size: 3rem; padding: 1em"
        >{{ username }}
        </v-card-title>
        <v-row
            align="center"
            justify="center"
        >
          <v-col>
            <v-img
                max-width="500px"
                :src=avatarURL
                style="margin: 0 auto; border-radius: 50%"
            ></v-img>
          </v-col>
          <v-col>
            <div
                style="margin: 1em"
            >
              <h1>{{ fullName }}</h1>
              <p v-if="bio != null">{{ bio }}</p>
              <p style="display: inline-block; margin-right: 1em">
                <v-icon
                    medium
                >mdi-account-group
                </v-icon>
                {{ followers }} followers
              </p>
              <p style="display: inline-block; margin-right: 1em">
                <v-icon
                    medium
                >mdi-account-supervisor
                </v-icon>
                {{ following }} following
              </p>
              <p style="display: inline-block; margin-right: 1em">
                <v-icon
                    medium
                >mdi-source-repository
                </v-icon>
                {{ numRepos }} repositories
              </p>
              <img :src="statURL" />
            </div>
          </v-col>
        </v-row>
        <v-container>
          <v-row style="margin: 1em">
            <v-col>
              <div style="text-align: center">
                <h3>Languages in owned repositories (measured in bytes):</h3>
                <pie-chart :data="languages" :donut="true"></pie-chart>
              </div>
            </v-col>
            <v-col>
              <div style="text-align: center">
                <h3>Commits made in owned repositories:</h3>
                <pie-chart :data="repoCommits" :donut="true"></pie-chart>
              </div>
            </v-col>
            <v-col>
              <div style="text-align: center">
                <h3>Recent Commits:</h3>
                <column-chart :data="recentActivity"></column-chart>
              </div>
            </v-col>
          </v-row>
          <v-row style="margin: 3em 0 0 0">
            <v-col>
              <div style="text-align: center">
                <h3>Commits in 2020:</h3>
                <vue-chart-heatmap :selector="'contributionHeatMap'" :entries="contributionsData" :locale="locale" :color-range="colourRange"
                                 :tooltip-enabled="false"></vue-chart-heatmap>
              </div>
            </v-col>
          </v-row>
          <v-row>

          </v-row>
          <v-row style="margin: 1em">
            <v-col>
              <h3 style="text-align: center">Click below to learn more about a repository:</h3>
              <div style="display: flex; flex-direction: row; align-items: center; flex-wrap: wrap">
                <v-card v-on:click="searchRepo(repo.full_name)" v-for="repo in repos" :key="repo"
                        elevation="2"
                        style="margin: 1rem; text-align:center; width: 30%; min-width: 200px">
                  <v-card-title
                      class="justify-center"
                  >{{ repo.full_name }}
                  </v-card-title>
                  <v-card-text
                      align="center"
                  >{{ repo.description }}
                  </v-card-text>
                </v-card>
              </div>
            </v-col>
          </v-row>
        </v-container>
      </v-card>
    </v-container>
  </div>
</template>

<script>
import axios from "axios";
import Unknown from "./Unknown";
import VueChartHeatmap from 'vue-chart-heatmap';

export default {
  name: 'User',

  components: {
    Unknown,
    VueChartHeatmap,
  },

  data: () => ({
    display: true,
    userData: "",

    // Extracted user data
    username: "",
    avatarURL: "",
    fullName: "",
    bio: "",
    followers: "",
    following: "",

    numRepos: "",
    repos: "",

    languages: [],
    recentActivity: [],
    repoCommits: [],
    contributionChart: "",
    contributionsData: [],
    locale: {
      months: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
      days: ['S', 'M', 'T', 'W', 'T', 'F', 'S'],
      No: 'No',
      on: 'on',
      Less: 'Less',
      More: 'More'
    },
    colourRange: ["#ededed", "#16f529"],
    statURL: "",
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
      this.userData = "";
      this.username = "";
      this.avatarURL = "";
      this.fullName= "";
      this.bio = "";
      this.followers = "";
      this.following = "";
      this.numRepos = "";
      this.repos = "";
      this.languages = [];
      this.recentActivity = [];
      this.repoCommits = [];
      this.contributionChart = "";
      this.contributionsData = [];
      this.getUserData();
      this.getRecentActivity();
    },

    getUserData() {
      let baseURL = "https://api.github.com";
      let username = this.toSearch;
      const urlToQuery = baseURL + "/users/" + username;
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            this.display = false;
            this.userData = response;
            if(this.userData.data.type === "User") {
              this.display = true;
            }
            this.extractUserInfo();
            this.getRepoData();
          })
          .catch(error => {
            this.display = false;
            console.log(error);
          })
    },

    extractUserInfo() {
      this.username = this.userData.data.login;
      this.avatarURL = this.userData.data.avatar_url;
      this.fullName = this.userData.data.name;
      this.bio = this.userData.data.bio;
      this.followers = this.userData.data.followers;
      this.following = this.userData.data.following;
      this.statURL = "https://github-readme-stats.vercel.app/api?username=" + this.username;
      this.getContributions();
    },


    getRecentActivity() {
      let baseURL = "https://api.github.com";
      let username = this.toSearch;
      const urlToQuery = baseURL + "/users/" + username + "/events/public";
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            let commitsMap = new Map();
            for (let i = 0; i < response.data.length; i++) {
              let event = response.data[i];
              if (event.type === "PushEvent") {
                let commits = event.payload.commits;
                for (let j = 0; j < commits.length; j++) {
                  if (commits[j].author.name === this.fullName) {
                    if (commitsMap.has(event.repo.name)) {
                      let num = commitsMap.get(event.repo.name);
                      commitsMap.set(event.repo.name, num + 1);
                    } else {
                      commitsMap.set(event.repo.name, 1);
                    }
                  }
                }
                this.recentActivity = [];
                for (const [key, value] of commitsMap) {
                  this.recentActivity.push([key, value]);
                }
              }
            }
          })
          .catch(error => {
            this.display = false;
            console.log(error);
          })
    },

    getRepoData() {
      const urlToQuery = this.userData.data.repos_url;
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            this.numRepos = response.data.length;
            this.repos = response.data;
            this.extractLanguages();
            setTimeout(this.extractRepoCommits, 2000);
          })
          .catch(error => {
            this.display = false;
            console.log(error);
          })
    },

    extractRepoCommits() {
      let baseURL = "https://api.github.com";
      let username = this.toSearch;
      let repoCommits = new Map();
      for (let i = 0; i < this.repos.length; i++) {
        let repoName = this.repos[i].name;
        const urlToQuery = baseURL + "/repos/" + username + "/" + repoName + "/stats/contributors";
        axios
            .get(urlToQuery, {
              headers: {
                authorization: "token " + this.token
              },
              timeout: 10000
            })
            .then(response => {
              let commitStats = response.data;
              for (let j = 0; j < commitStats.length; j++) {
                let contributor = commitStats[j];
                if (contributor !== undefined && contributor.author.login === username) {
                  repoCommits.set(repoName, contributor.total);
                }
              }
              this.repoCommits = [];
              for (const [key, value] of repoCommits) {
                if (key !== "undefined" && key !== undefined) {
                  this.repoCommits.push([key, value]);
                }
              }
            })
            .catch(error => {
              this.display = false;
              console.log(error);
            })
      }
    },

    extractLanguages() {
      let baseURL = "https://api.github.com";
      let username = this.toSearch;
      let languages = new Map();
      for (let i = 0; i < this.repos.length; i++) {
        const urlToQuery = baseURL + "/repos/" + username + "/" + this.repos[i].name + "/languages";
        axios
            .get(urlToQuery, {
              headers: {
                authorization: "token " + this.token
              },
              timeout: 10000
            })
            .then(response => {
              for (const [key, value] of Object.entries(response.data)) {
                if (key !== undefined && key !== "undefined") {
                  if (languages.has(key)) {
                    let num = languages.get(key);
                    languages.set(key, num + value);
                  } else {
                    languages.set(key, value);
                  }
                }
              }
              this.languages = [];
              for (const [key, value] of languages) {
                if (key !== "undefined" && key !== undefined) {
                  this.languages.push([key, value]);
                }
              }
            })
            .catch(error => {
              this.display = false;
              console.log(error);
            })
      }
    },

    async getContributions() {
      const headers = {
        'Authorization': `bearer ${this.token}`,
      }
      const body = {
        "query": `query {
            user(login: "${this.username}") {
              name
              contributionsCollection {
                contributionCalendar {
                  totalContributions
                  weeks {
                    contributionDays {
                      color
                      contributionCount
                      date
                      weekday
                    }
                    firstDay
                  }
                }
              }
            }
          }`
      }
      const response = await fetch('https://api.github.com/graphql', {
        method: 'POST',
        body: JSON.stringify(body),
        headers: headers
      })
      const data = await response.json()
      let contribs = data.data.user.contributionsCollection.contributionCalendar.weeks;
      this.contributionsData = [];
      for (let i = 0; i < contribs.length; i++) {
        let week = contribs[i];
        let days = week.contributionDays;
        for (let j = 0; j < days.length; j++) {
          let day = days[j];
          let created_at = day.date;
          let counting = day.contributionCount;
          this.contributionsData.push({counting, created_at});
        }
      }
    },

    searchRepo(repo) {
      this.$emit("searchRepo", repo);
    },
  }
}
</script>
