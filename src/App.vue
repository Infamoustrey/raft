<template>
  <v-app>
    <v-toolbar color="primary white--text" app>
      <v-icon color="white">fa-life-ring</v-icon>
      <v-toolbar-title v-text="title"></v-toolbar-title>
      <v-spacer></v-spacer>
    </v-toolbar>
    <v-content>
      <router-view/>
    </v-content>
    <v-footer app>
      <span>&copy; 2017</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      title: 'Raft'
    }
  },
  name: 'App'
}
</script>
