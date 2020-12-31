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
        >{{ repoName }}
        </v-card-title>
        <v-card-text
            align="center"
        >{{ repoDescription }}
        </v-card-text>
        <v-row>
          <v-col>
            <h3>Commits by contributors over time:</h3>
            <line-chart :data="chartData"></line-chart>
          </v-col>
        </v-row>
        <v-row
        >
          <v-col>
            <h3>Top 5 Contributors (commits):</h3>
            <v-card v-on:click="searchUser(user.login)" v-for="(user, index) in contributors" :key="user"
                    elevation="2"
                    style="padding: 1rem; margin: 0.5rem">
              <div style="display: flex; flex-direction: row; align-items: center">
                <h4 style="margin: 0">{{ index + 1 }}.</h4>
                <img style="margin: 0 1em;width: 3rem; border-radius: 50%" :src=user.image />
                <p style="margin: 0"><strong>{{ user.login }}:</strong> {{ user.num }} contributions</p>
              </div>
            </v-card>
          </v-col>
          <v-col>
            <h3>Top 5 Contributors (additions and deletions):</h3>
            <v-card v-on:click="searchUser(user.name)" v-for="(user, index) in additionsDeletions" :key="user"
                    elevation="2"
                    style="padding: 1rem; margin: 0.5rem">
              <div style="display: flex; flex-direction: row; align-items: center">
                <h4 style="margin: 0">{{ index + 1 }}.</h4>
                <img style="margin: 0 1em;width: 3rem; border-radius: 50%" :src=user.image />
                <p style="margin: 0"><strong>{{ user.name }}:</strong> {{ user.additions }} additions and
                  {{ user.deletions }} deletions</p>
              </div>
            </v-card>
          </v-col>
          <v-col>
            <h3>Top 5 Commenters:</h3>
            <v-card v-on:click="searchUser(user.login)" v-for="(user, index) in commenters" :key="user"
                    elevation="2"
                    style="padding: 1rem; margin: 0.5rem">
              <div style="display: flex; flex-direction: row; align-items: center">
                <h4 style="margin: 0">{{ index + 1 }}.</h4>
                <img style="margin: 0 1em;width: 3rem; border-radius: 50%" :src=user.data.image />
                <p style="margin: 0"><strong>{{ user.login }}:</strong> {{ user.data.num }} comments</p>
              </div>
            </v-card>
          </v-col>
        </v-row>
        <v-row>
          <v-col>
            <v-card
                elevation="2"
                style="padding: 1rem; margin: 0.5rem"
            >
              <v-card-title
                  class="justify-center"
                  style="font-size: 2rem; padding-top: 1em"
              >Repository Heroes
              </v-card-title>
              <v-card-text
                  align="center"
              >"Hero Rating" is a metric (devised by Eoin Pinaqui for the CSU33012 Software Engineering module) that
                can be used to rank contributors based on their involvement in a repository.
              </v-card-text>
              <v-card v-on:click="searchUser(user[0])" v-for="user in shortHeroRating" :key="user"
                      elevation="2"
                      class="justify-center"
                      style="padding: 1rem; margin: 2em auto; text-align: center; max-width: 500px">
                <div style="display: flex; flex-direction: row; align-items: center">
                  <v-icon>mdi-cards-heart</v-icon>
                  <img style="margin: 0 1em; width: 10em; border-radius: 50%" :src=user[2] />
                  <p style="margin: 0; font-size: 2em"><strong>{{ user[0] }}</strong></p>
                </div>
              </v-card>
            </v-card>
          </v-col>
        </v-row>
        <v-row style="margin: 1em">
          <v-col>
            <h3 style="text-align: center">Click below to learn more about a contributor (sorted by hero rating):</h3>
            <div style="display: flex; flex-direction: row; align-items: center; flex-wrap: wrap">
              <v-card v-on:click="searchUser(user[0])" v-for="(user, index) in heroRating" :key="user"
                      elevation="2"
                      style="padding: 1rem; margin: 0.5rem; width: 19%; text-align: center">
                <div style="display: flex; flex-direction: row; align-items: center">
                  <h4 style="margin: 0">{{ index + 1 }}.</h4>
                  <img style="margin: 0 1em;width: 3rem; border-radius: 50%" :src=user[2] />
                  <p style="margin: 0"><strong>{{ user[0] }}</strong></p>

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
    repoData: "",

    /* Extracted repo data */
    repoName: "",
    repoDescription: "",
    commitData: [],
    chartData: [],
    cancel: false,
    contributors: [],
    allContributors: [],
    allAdditionsDeletions: [],
    additionsDeletions: [],
    commenters: [],
    allCommenters: [],
    heroRating: [],
    shortHeroRating: [],
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
      this.repoName = "";
      this.repoDescription = "";
      this.commitData = [];
      this.chartData = [];
      this.cancel = false;
      this.contributors = [];
      this.allContributors = [];
      this.allAdditionsDeletions = [];
      this.additionsDeletions = [];
      this.commenters = [];
      this.allCommenters = [];
      this.heroRating = [];
      this.getRepoInfo();
    },

    getRepoInfo() {
      this.getRepoData();
      this.getContributors(1);
      this.getCommitInfo();
      this.getComments();
      let ref = this;
      setTimeout(function() {
        ref.calculateHeroRatings();
      }, 10000);
    },

    getRepoData() {
      let baseURL = "https://api.github.com";
      let repo = this.toSearch;
      const urlToQuery = baseURL + "/repos/" + repo;
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            let repoData = response.data;
            this.repoName = repoData.name;
            this.repoDescription = repoData.description;
          })
          .catch(error => {
            this.display = false;
            console.log(error);
          })
    },

    getContributors(page) {
      let baseURL = "https://api.github.com";
      let repo = this.toSearch;
      const urlToQuery = baseURL + "/repos/" + repo + "/contributors?per_page=100&page=" + page;
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            let contribs = response.data;
            this.contributors = [];
            for (let i = 0; i < contribs.length; i++) {
              let x = contribs[i];
              let login = x.login;
              let num = x.contributions
              let image = x.avatar_url;
              this.allContributors.push({login, num, image});
            }
            if(contribs.length >= 100) {
              this.getContributors(page + 1);
            } else {
              this.contributors = this.allContributors.slice(0, 5);
            }
          })
          .catch(error => {
            this.display = false;
            console.log(error);
          })
    },

    getCommitInfo() {
      let baseURL = "https://api.github.com";
      let repo = this.toSearch;
      const urlToQuery = baseURL + "/repos/" + repo + "/stats/contributors";
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            let contributorsInfo = response.data;
            contributorsInfo.reverse();
            contributorsInfo = contributorsInfo.slice(0, 25);

            this.chartData = [];
            for (let i = 0; i < contributorsInfo.length; i++) {
              let x = contributorsInfo[i];
              let name = x.author.login;
              let weeks = x.weeks;
              let data = {};

              let additions = 0;
              let deletions = 0;

              for (let j = 0; j < weeks.length; j++) {
                let week = weeks[j];
                if (week.c > 0) {
                  let dateObject = new Date(week.w * 1000);
                  let dateFormat = dateObject.toISOString();
                  let date = dateFormat.substring(0, 10);
                  data[date] = week.c;
                }
                additions += week.a;
                deletions += week.d;
              }
              this.chartData.push({name, data});
              let image = x.author.avatar_url;
              this.allAdditionsDeletions.push({name, additions, deletions, image});
            }
            this.allAdditionsDeletions.sort((a, b) => (a.additions + a.deletions > b.additions + b.deletions) ? -1 : (b.additions + b.deletions > a.additions + a.deletions) ? 1 : 0);
            this.additionsDeletions = this.allAdditionsDeletions.slice(0, 5);
          })
          .catch(error => {
            this.display = false;
            console.log(error);
          })
    },

    getComments() {
      let baseURL = "https://api.github.com";
      let repo = this.toSearch;
      const urlToQuery = baseURL + "/repos/" + repo + "/comments";
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            let comments = response.data;
            let userComments = new Map();
            for (let i = 0; i < comments.length; i++) {
              let comment = comments[i];
              if (userComments.has(comment.user.login)) {
                let old = userComments.get(comment.user.login);
                let num = old.num + 1;
                let image = old.image;
                userComments.set(comment.user.login, {num, image});
              } else {
                let num = 1;
                let image = comment.user.avatar_url;
                userComments.set(comment.user.login, {num, image});
              }
            }
            for (const [login, data] of userComments) {
              this.allCommenters.push({login, data});
            }
            this.allCommenters.sort((a, b) => (a.data.num > b.data.num) ? -1 : (b.data.num > a.data.num) ? 1 : 0);
            this.commenters = this.allCommenters.slice(0, 5);
          })
          .catch(error => {
            this.display = false;
            console.log(error);
          })
    },

    calculateHeroRatings() {
      console.log(this.allContributors);
      if(this.allContributors.length === 1) {
        this.shortHeroRating = [[this.contributors[0].login, 10, this.contributors[0].image]];
        return;
      }

      let commits = this.allContributors;
      let additions = this.allAdditionsDeletions;
      let commenters = this.allCommenters;

      let totalCommits = 0;
      let totalAdditions = 0;
      let totalComments = 0;


      for (let i = 0; i < commits.length; i++) {
        totalCommits += commits[i].num;
      }


      for (let i = 0; i < additions.length; i++) {
        totalAdditions += additions[i].additions + additions[i].deletions;
      }

      for(let i = 0; i < commenters.length; i++) {
        totalComments += commenters[i].data.num;
      }


      let heroMap = new Map();

      for(let i = 0; i < commits.length; i++) {
        heroMap.set(commits[i].login, [5 * (commits[i].num / totalCommits), commits[i].image]);
      }

      for(let i = 0; i < additions.length; i++) {
        if(heroMap.has(additions[i].name)) {
          let x = heroMap.get(additions[i].name);
          let y = 4 * ((additions[i].additions + additions[i].deletions) / totalAdditions);
          let z = [y + x[0], x[1]];
          heroMap.set(additions[i].name, z);
        }
      }

      for(let i = 0; i < commenters.length; i++) {
        if(heroMap.has(commenters[i].login)) {
          let x = heroMap.get(commenters[i].login);
          let y = (commenters[i].data.num) / totalComments;
          let z = [y + x[0], x[1]];
          heroMap.set(commenters[i].login, z);
        }
      }

      for(const [key, value] of heroMap) {
        this.heroRating.push([key, value[0], value[1]]);
      }

      this.heroRating.sort((a, b) => (a[1] > b[1]) ? -1 : (b[1] > a[1]) ? 1 : 0);
      this.shortHeroRating = this.heroRating.slice(0, 3);
    },

    searchUser(user) {
      this.$emit("searchUser", user);
    },
  },
}
</script>