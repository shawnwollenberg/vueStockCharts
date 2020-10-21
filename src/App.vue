<template>
  <v-app>
    <v-app-bar
      app
      color="#fff"
      
    >
    <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          src="@/assets/logo.png"
          transition="scale-transition"
        />
    </v-app-bar>

    <v-main style="background-color:#f2f2f2">
      <router-view/>
    </v-main>
  </v-app>
</template>

<script>
export default {
  name: 'App',
  data: () => ({
    //
  }),
};
</script>
