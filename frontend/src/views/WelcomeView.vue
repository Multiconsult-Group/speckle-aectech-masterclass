<template lang="html">
  <v-container
    fill-height
    class="home flex-column justify-center align-center primary--text"
  >
    <v-img src="@/assets/logo.png" max-height="140px" max-width="140px"></v-img>
    <h1>Welcome to the Speckle Diff Visualizer</h1>
    <p>This app generates a heat map diff of changed geometry.</p>
    <v-alert type="info" text color="primary">
      This app is part of the AEC Tech Masterclass
    </v-alert>
    <p class="grey--text">Please log in to access you Speckle data.</p>
  </v-container>
</template>
<script>
export default {
  name: "WelcomeView"
}
</script>
