<template>
  <v-app>

    <v-container>
      <v-row>
        <v-col>
          <v-text-field
            label="Search"
            v-model="tempSearch"
          ></v-text-field>
        </v-col>
        <v-col>
          <v-select
            :items="dropdown"
            v-model="dropdownSelected"
            label="Where?"
          ></v-select>
          <div style="text-align: right">
            <v-btn
                v-on:click="getData"
                color="primary"
                elevation="4"
                large
            >Let's go!</v-btn>
          </div>
        </v-col>
      </v-row>
    </v-container>

    <v-main>
      <User v-if="displayUser && user_data != 'unknown'" v-bind:userData="user_data" />
    </v-main>
  </v-app>
</template>

<script>
import User from './components/User.vue';
import axios from "axios";

export default {
  name: 'App',

  components: {
    User
  },

  data: () => ({

    // variables for searching
    tempSearch: "",
    search: "",
    dropdown: ["Users", "Repositories", "Organisations"],
    dropdownSelected: "",

    // booleans for conditional rendering
    displayUser: false,
    displayRepo: false,
    displayOrg: false,
    displayUnknown: false,

    // user variables
    user_data: ""
  }),

  methods: {
    getData: function() {
      const baseURL = "https://api.github.com";

      this.search = this.tempSearch;
      this.user_data = "unknown";

      if(this.dropdownSelected === "Users") {
        const urlToQuery = baseURL + "/users/" + this.search;
        axios
            .get(urlToQuery)
            .then(response => (this.user_data = response))
            .catch(error => alert(error));

        this.displayUser = true;
        this.displayRepo = false;
        this.displayOrg = false;
        this.displayUnknown = false;
      } else if(this.dropdownSelected === "Repositories") {
        this.displayUser = false;
        this.displayRepo = true;
        this.displayOrg = false;
        this.displayUnknown = false;
      } else if(this.dropdownSelected === "Organisations") {
        this.displayUser = false;
        this.displayRepo = false;
        this.displayOrg = true;
        this.displayUnknown = false;
      } else {
        this.displayUser = false;
        this.displayRepo = false;
        this.displayOrg = false;
        this.displayUnknown = false;
      }
    }
  }
};
</script>
