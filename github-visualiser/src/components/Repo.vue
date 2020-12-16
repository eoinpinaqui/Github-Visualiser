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
            align="center"
            justify="center"
        >
          <ol>
            <li v-for="commit in commitData" :key="commit">
              {{ commit }}
            </li>
          </ol>
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
    commitData: ""
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
            this.display = true;
            this.repoData = response.data;
            this.repoName = this.repoData.name;
            this.repoDescription = this.repoData.description;
            this.getCommitInfo(this.repoData.commits_url);
          })
          .catch(error => {
            this.display = false;
            alert(error);
          })
    },

    getCommitInfo(commitsURL) {
      let urlToQuery = commitsURL.substring(0, commitsURL.length - 6);
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            this.commitData = response.data;
            if(this.commitData.length > 0) {
              console.log("Page 1: " + this.commitData.length);
              this.getNextPage(urlToQuery, 2);
            }
          })
          .catch(error => {
            this.display = false;
            alert(error);
          })
    },

    getNextPage(commitURL, page) {
      let urlToQuery = commitURL + "?page=" + page;
      axios
          .get(urlToQuery, {
            headers: {
              authorization: "token " + this.token
            },
            timeout: 10000
          })
          .then(response => {
            let moreCommits = response.data;
            if(moreCommits.length > 0) {
              console.log("Page " + page + ": " + moreCommits.length);
              this.commitData = this.commitData.concat(moreCommits);
              this.getNextPage(commitURL, page + 1);
            } else {
              console.log("Length: " +this.commitData.length);
            }
          })
          .catch(error => {
            this.display = false;
            alert(error);
          })
    }
  }
}
</script>