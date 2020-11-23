<template>
  <v-container>
    <v-card
        elevation="2"
        style="padding: 3rem"
    >
      <v-card-title
          class="justify-center"
          style="font-size: 2rem; padding: 1em"
      >I'm sorry, we couldn't find "{{search}}" in "{{category}}"
      </v-card-title>
      <v-img
          max-width="300px"
          src="https://svgsilh.com/svg/2831367.svg"
          style="margin: 1rem auto"
      ></v-img>
    </v-card>
  </v-container>
</template>

<script>
export default {
  name: "Unknown",

  props: {
    search: String,
    category: String,
  }
}
</script>