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
        </v-row>
        <v-row

        >
          <v-col>
            <h3>Commits by Top 50 Contributors over time:</h3>
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

      let baseURL = "https://api.github.com";
      let repo = this.toSearch;
      this.commitData = [];
      const urlToQuery = baseURL + "/repos/" + repo;
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            this.display = true;
            this.repoData = response.data;
            this.repoName = this.repoData.name;
            this.repoDescription = this.repoData.description;

            let urlToQuery = this.repoData.commits_url.substring(0, this.repoData.commits_url.length - 6);
            this.cancel = false;
            this.getNextPageOfCommits(urlToQuery, 1, 100);
            setTimeout('this.contributors.sort((a, b) => (a.num > b.num) ? 1 : ((b. num > a.num) ? -1 : 0))', 1000);
          })
          .catch(error => {
            this.display = false;
            alert(error);
          })
    },

    getNextPageOfCommits(commitURL, page, perPage) {
      if (this.cancel === true) {
        console.log("CANCELLING");
        this.cancel = false;
      } else {
        let urlToQuery = commitURL + "?page=" + page + "&per_page=" + perPage;
        axios
            .get(urlToQuery, {
              headers: {
                authorization: "token " + this.token
              },
              timeout: 10000
            })
            .then(response => {
              let moreCommits = response.data;
              if (moreCommits.length > 0) {
                this.commitData = this.commitData.concat(moreCommits);
                if (page % 10 === 0 || page === 1) {
                  console.log("Page " + page + ": UPDATING CHART");
                  this.extractChartData()
                }
                this.getNextPageOfCommits(commitURL, page + 1, perPage);
              } else {
                this.extractChartData()
              }
            })
            .catch(error => {
              this.display = false;
              alert(error);
            })
      }
    },

    extractChartData() {
      let commitInfo = [];
      for (let i = 0; i < this.commitData.length; i++) {
        let commiti = this.commitData[i];
        let name = commiti.commit.author.name;
        let date = commiti.commit.author.date;
        commitInfo.push({name, date});
      }

      let userMap = new Map();
      for (let i = 0; i < commitInfo.length; i++) {
        let user = commitInfo[i].name;
        let date = commitInfo[i].date;
        if (userMap.has(user)) {
          let commitDates = userMap.get(user);
          commitDates.push(date);
          userMap.set(user, commitDates);
        } else {
          userMap.set(user, []);
          let commitDates = userMap.get(user);
          commitDates.push(date);
          userMap.set(user, commitDates);
        }
      }

      let chart = [];

      for (const [name, dates] of userMap) {
        let dateMap = new Map();
        for (let i = 0; i < dates.length; i++) {
          let commitDate = dates[i];
          commitDate = commitDate.substring(0, 10);
          if (dateMap.has(commitDate)) {
            let numCommits = dateMap.get(commitDate);
            dateMap.set(commitDate, numCommits + 1);
          } else {
            dateMap.set(commitDate, 1);
          }
        }

        let data = {};
        for (const [date, num] of dateMap) {
          data[date] = num;
        }
        chart.push({name, data})
      }
      this.chartData = chart.slice(0, 50);
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
            console.log(contribs);
            this.contributors = [];
            for (let i = 0; i < contribs.length && i < 10; i++) {
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