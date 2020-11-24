<template>
  <Unknown v-if="!display" :search="toSearch" category="Users"></Unknown>
  <v-container v-else-if="display">
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
            <p>
              <v-icon
                  medium
              >mdi-account-group
              </v-icon>
              {{ followers }} followers
            </p>
            <p>
              <v-icon
                  medium
              >mdi-account-supervisor
              </v-icon>
              {{ following }} following
            </p>
            <p>
              <v-icon
                  medium
              >mdi-source-repository
              </v-icon>
              {{ repos }} repositories
            </p>
          </div>
        </v-col>
      </v-row>
      <v-container>
        <v-row style="margin: 1em">
          <v-col>
            <h3>Languages in owned repositories (measured in bytes):</h3>
            <pie-chart :data="languages" :donut="true"></pie-chart>
          </v-col>
          <v-col>
            <h3>Commit History:</h3>
            <line-chart :data="{'2017-05-13': 2, '2017-05-14': 5}"></line-chart>
          </v-col>
          <v-col>
            <h3>Recent Commits</h3>
            <column-chart :data="recentActivity"></column-chart>
          </v-col>
        </v-row>
      </v-container>
    </v-card>
  </v-container>
</template>

<script>
import axios from "axios";
import Unknown from "./Unknown";

export default {
  name: 'User',

  components: {
    Unknown
  },

  data: () => ({
    display: false,
    userData: "",

    // Extracted user data
    username: "",
    avatarURL: "",
    fullName: "",
    bio: "",
    followers: "",
    following: "",
    repos: "",
    repoData: "",
    languages: "",
    commitData: "",
    recentActivity: "",
  }),

  props: {
    toSearch: String,
    token: String,
  },

  methods: {
    search() {
      this.display = true;
      let baseURL = "https://api.github.com";
      let usernameString = this.toSearch;
      let urlToQuery = baseURL + "/users/" + usernameString;

      // Fetch general data about user from the Github API
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            this.userData = response;
            this.username = this.userData.data.login;
            this.avatarURL = this.userData.data.avatar_url;
            this.fullName = this.userData.data.name;
            this.bio = this.userData.data.bio;
            this.followers = this.userData.data.followers;
            this.following = this.userData.data.following;

            const eventURL = baseURL + "/users/" + usernameString + "/events/public";
            axios
                .get(eventURL)
                .then(response => {
                  this.commitData = response.data;
                  let commitsMap = new Map();
                  for(let i = 0; i < response.data.length; i++) {
                    let event = response.data[i];
                    if(event.type === "PushEvent") {
                      let commits = event.payload.commits;
                      for(let j = 0; j < commits.length; j++) {
                        if(commits[j].author.name === this.fullName) {
                          if(commitsMap.has(event.repo.name)) {
                            let num = commitsMap.get(event.repo.name);
                            commitsMap.set(event.repo.name, num + 1);
                          } else {
                            commitsMap.set(event.repo.name, 1);
                          }
                        }
                      }
                    }
                  }
                  this.recentActivity = [];
                  for (const [key, value] of commitsMap) {
                    this.recentActivity.push([key, value]);
                  }
                })

            let reposURL = this.userData.data.repos_url;
            axios
                .get(reposURL, {
                  headers: {
                    authorization: "token " + this.token
                  },
                  timeout: 10000
                })
                .then(response => {
                  this.repos = response.data.length;

                  let languages = new Map();
                  let repoCommits = new Map();
                  for (let i = 0; i < response.data.length; i++) {
                    let repoName = response.data[i].name;
                    const commitsURL = baseURL + "/repos/" + usernameString + "/" + repoName + "/stats/contributors";
                    axios
                        .get(commitsURL)
                        .then(response => {
                          let commitStats = response.data;
                          for(let j = 0; j < commitStats.length; j++) {
                            let contributors = commitStats[i];
                            for(let k = 0; k < contributors.length; k++) {
                              let contributor = contributors[k];
                              if(contributor.author.login === usernameString) {
                                repoCommits.set(repoName, contributor.total);
                              }
                            }
                          }
                          console.log(response.data);
                        })



                    const languagesURL = baseURL + "/repos/" + usernameString + "/" + repoName + "/languages";
                    axios
                        .get(languagesURL, {
                          headers: {
                            authorization: "token" + this.token
                          },
                          timeout: 10000
                        })
                        .then(response => {
                          for (const [key, value] of Object.entries(response.data)) {
                            if (key !== undefined && key !== "undefined") {
                              if(languages.has(key)) {
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
                          console.log(error);
                          this.display = false;
                        });

                  }
                })
                .catch(error => {
                  console.log(error);
                  this.display = false;
                });
          })
          .catch(error => {
            console.log(error);
            this.display = false;
          });
    }
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
  }
}
</script>
