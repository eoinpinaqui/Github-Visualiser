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
        >
          <v-col>

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
      this.getRepoInfo();
    },


    searchUser(user) {
      this.$emit("searchUser", user);
    },
  },
}
</script>