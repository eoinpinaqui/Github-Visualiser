<template>
  <v-container>
    <v-card
        elevation="2"
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
            <h3>Programming Languages (measured in bytes written):</h3>
            <pie-chart :data="languages"></pie-chart>
          </v-col>
          <v-col>
            <p>This is it isnt it.</p>
          </v-col>
        </v-row>
      </v-container>
    </v-card>
  </v-container>
</template>

<script>
import axios from "axios";

export default {
  name: 'User',

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

  }),

  props: {
    toSearch: String,
  },

  methods: {
    search() {
      this.display = false;
      let baseURL = "https://api.github.com";
      let usernameString = this.toSearch;
      let urlToQuery = baseURL + "/users/" + usernameString;
      axios
          .get(urlToQuery)
          .then(response => {
            this.userData = response;
            this.username = this.userData.data.login;
            this.avatarURL = this.userData.data.avatar_url;
            this.fullName = this.userData.data.name;
            this.bio = this.userData.data.bio;
            this.followers = this.userData.data.followers;
            this.following = this.userData.data.following;

            let reposURL = this.userData.data.repos_url;
            axios
                .get(reposURL)
                .then(response => {
                  this.repos = response.data.length;
                  let languages = new Map();
                  for (let i = 0; i < response.data.length; i++) {
                    let repoName = response.data[i].name;
                    const languagesURL = baseURL + "/repos/" + usernameString + "/" + repoName + "/languages";
                    axios
                        .get(languagesURL)
                        .then(response => {
                          for (const [key, value] of Object.entries(response.data)) {
                            if(key !== undefined && key !== "undefined"){
                              languages.set(key, value);
                            }
                          }
                          this.languages = [];
                          for (const [key, value] of languages) {
                            if (key !== "undefined" && key !== undefined) {
                              this.languages.push([key, value]);
                            }
                          }
                          console.log(this.languages);
                        })
                  }
                })
          })
          .catch(error => alert(error));

    }
  },

  created() {
    this.search();
  },
  watch: {
    toSearch: function () {
      this.search();
    }
  }
}
</script>
