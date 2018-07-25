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
    <v-footer>
      <p class="ma-0 text-xs-center" style="width:100%;">&copy; {{currentYear}} Trey Foster</p>
    </v-footer>
  </v-app>
</template>

<script>
import moment from 'moment'
export default {
  data () {
    return {
      title: 'Raft'
    }
  },
  computed: {
    currentYear(){return moment().format('YYYY')}
  },
  name: 'App'
}
</script>
