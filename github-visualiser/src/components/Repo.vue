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
        <v-row
        >
          <v-col>
            <h3>Top {{ numContributors }} Contributors:</h3>
            <v-card v-for="(user, index) in contributors" :key="user"
                    elevation="2"
                    style="padding: 1rem; margin: 0.5rem">
              <div style="display: flex; flex-direction: row; align-items: center">
                <h4 style="margin: 0">{{index + 1}}</h4>
                <img style="margin: 0 1em;width: 3rem; border-radius: 50%" :src=user.image />
                <p style="margin: 0"><strong>{{user.login}}:</strong> {{user.num}} contributions</p>
              </div>
            </v-card>
          </v-col>
          <v-col>

          </v-col>
          <v-col>

          </v-col>
        </v-row>
        <v-row

        >
          <v-col>
            <h3>Commits by user over time:</h3>
            <line-chart :data="chartData"></line-chart>
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
    numContributors: 0,
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
      this.cancel = true;
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
      this.getRepoInfo();
    },

    getRepoInfo() {
      this.getContributors();
      this.getCommitInfo();
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

            this.chartData = [];
            for(let i = 0; i < contributorsInfo.length; i++) {
              let x = contributorsInfo[i];
              let name = x.author.login;
              let weeks = x.weeks;
              let data = {};
              for(let j = 0; j < weeks.length; j++) {
                let week = weeks[j];
                if(week.c > 0) {
                  let dateObject = new Date(week.w * 1000);
                  let dateFormat = dateObject.toISOString();
                  let date = dateFormat.substring(0, 10);
                  data[date] = week.c;
                }
              }
              this.chartData.push({name, data});
            }
          })
          .catch(error => {
            this.display = false;
            alert(error);
          })
    },

    getContributors() {
      let baseURL = "https://api.github.com";
      let repo = this.toSearch;
      const urlToQuery = baseURL + "/repos/" + repo + "/contributors";
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
            for (let i = 0; i < contribs.length && i < 5; i++) {
              let x = contribs[i];
              this.getUserImage(x.url, x.login, x.contributions);
              this.numContributors = i + 1;
            }
          })
          .catch(error => {
            this.display = false;
            alert(error);
          })
    },

    getUserImage(urlToQuery, login, num) {
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            let image = response.data.avatar_url;
            this.contributors.push({login, num, image});
          })
          .catch(error => {
            this.display = false;
            alert(error);
          })
    },
  },
}
</script>