<template>
  <v-app>
    <v-container>
      <h1
        style="text-align: center; font-size: 3.5em"
      > Github Visualiser</h1>
    </v-container>
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
      <User v-if="displayUser" :toSearch="search" token="9b6a46190d38d61f8af4871f41a7df0a9627c73f" />
    </v-main>
  </v-app>
</template>

<script>
import User from './components/User.vue';

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
  }),

  methods: {
    getData: function() {
      this.search = this.tempSearch;
      if(this.dropdownSelected === "Users") {
        this.displayUser = true;
        this.displayRepo = false;
        this.displayOrg = false;
      } else if(this.dropdownSelected === "Repositories") {
        this.displayUser = false;
        this.displayRepo = true;
        this.displayOrg = false;
      } else if(this.dropdownSelected === "Organisations") {
        this.displayUser = false;
        this.displayRepo = false;
        this.displayOrg = true;
      } else {
        this.displayUser = false;
        this.displayRepo = false;
        this.displayOrg = false;
      }
    }
  }
};
</script>

<style>
* {
  font-family: "Roboto", sans-serif;
  color: #262626;
}

</style>
