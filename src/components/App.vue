<template>
  <v-app>
    <v-navigation-drawer temporary hide-overlay fixed v-model="drawer" class="text-xs-center" app>
      <v-list dense>
        <v-list-item v-for="srv in servers" :key="srv.id" :to="`/${srv.id}/clients`">
          <v-list-item-action>
            <v-icon>storage</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>{{srv.name}}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar fixed app dark elevate-on-scroll color="" style="background-color: #0cbd7c;">
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" v-if="servers.length > 1"/>
      <v-toolbar-title>{{(server ? server.name : '')}}</v-toolbar-title>
      <v-spacer/>
      <v-flex xs2>
        <v-text-field
          class="mt-5"
          prepend-inner-icon="search"
          single-line
          :append-icon="(search.length ? 'clear' : '')"
          ref="searchField"
          @click:append="search = ''"
          @keyup.esc="search = ''"
          v-model="search"/>
      </v-flex>
    </v-app-bar>
    <v-main style="background-color: #000;">
      <v-tabs v-model="tabState" centered>
        <v-tabs-slider color="green"/>
        <v-tab to="clients" replace>
          <i class="fas fa-lg fa-laptop"/>
          &nbsp;Clients
        </v-tab>
        <v-tab to="map" replace>
          <i class="fas fa-lg fa-map-marked"/>
          &nbsp;Map
        </v-tab>
        <v-tab to="events" replace>
          <i class="fas fa-lg fa-bell"/>
          &nbsp;Events
        </v-tab>
      </v-tabs>
      <clients v-if="tab === 'clients'"/>
      <locations v-if="tab === 'map'"/>
      <events v-if="tab === 'events'"/>

      <v-snackbar v-model="snack.visible" bottom :timeout="snack.timeout" :color="snack.color">
        {{snack.text}}
        <v-btn text @click="snack.visible = false">
          Close
        </v-btn>
      </v-snackbar>
    </v-main>
  </v-app>
</template>

<script>
import moment from 'moment'
import {mapState} from 'vuex'
import Clients from './Clients.vue'
import Events from './Events.vue'
import Locations from './Locations.vue'

export default {
  name: 'App',
  components: {
    Clients,
    Events,
    Locations
  },
  data() {
    return {
      snack: {
        visible: false,
        timeout: 3000,
        color: 'success',
        text: ''
      },
      search: '',
      drawer: false
    }
  },
  created() {
    window.addEventListener('keydown', this.onKeyDown)
  },
  computed: {
    tabState: {
      get() {
        return this.tab
      },
      set(value) {
        this.$store.commit('updateTab', value)
      }
    },
    ...mapState({
      tab: 'tab',
      event: 'event',
      serverTime: state => moment(state.serverTime * 1000).format(state.config.dateFormat),
      server: state => state.servers.find(srv => srv.id === state.server),
      servers: 'servers'
    })
  },
  methods: {
    notify(text, type) {
      this.snack.text = text
      this.snack.color = type
      this.snack.visible = true
    },
    onKeyDown(ev) {
      if (ev.keyCode === 70 && (ev.ctrlKey || ev.metaKey)) {
        ev.preventDefault()
        this.$refs.searchField.focus()
      }
    }
  },
  watch: {
    server() {
      this.drawer = false
      this.search = ''
    },
    event(value) {
      const map = new Map([['connect', 'success'], ['disconnect', 'error'], ['reconnect', 'info']])
      this.notify(`${value.node} ${value.event}ed`, map.get(value.event) || 'cyan darken-2')
    },
    search(value = '') {
      this.$store.commit('changeSearch', {text: value})
    }
  }
}
</script>
<style>

.v-slide-group__content {
    background-color: #1d2d1b;
}

</style>
