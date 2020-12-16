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
        >{{ toSearch }}
        </v-card-title>
        <v-row
            align="center"
            justify="center"
        >
          <p>{{ repoData }}</p>
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
    repoData: ""
  }),

  props: {
    toSearch: String,
    token: String,
  },

  created() {
    this.search();
  },

  methods: {
    search() {
      this.getRepoInfo();
    },

    getRepoInfo() {
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
            this.repoData = response;
          })
          .catch(error => {
            alert(error);
          })
    }
  }
}
</script>