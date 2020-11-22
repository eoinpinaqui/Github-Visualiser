<template>
    <v-container>
      <v-card
          elevation="2"
      >
        <v-card-title>{{userData.data.login}}</v-card-title>
        <v-row
          align="center"
          justify="center"
        >
          <v-col>
            <v-img
              max-width="500px"
              :src=userData.data.avatar_url
              style="margin: 0 auto; border-radius: 50%"
              ></v-img>
          </v-col>
          <v-col>
            <p>{{userData}}</p>
          </v-col>
        </v-row>
      </v-card>
    </v-container>
</template>

<script>
export default {
  name: 'User',
  props: {
    userData: Object
  }
}
</script>
