<template>
  <div>
    <b-container>
      <b-alert variant="success" :show="success.curTime" dismissable @dismissed="this.success.curTime = 0" @dismiss-count-down="countDownChanged">
        <p>{{success.message}}</p>
        <b-progress variant="success" :max="success.maxTime" :value="success.curTime" height="4px">
        </b-progress>
      </b-alert>
      <b-jumbotron fluid>
        <template slot="header">
          Create New Server
        </template>
        <template slot="lead">
          Spin up a new server.
        </template>
      </b-jumbotron>
      <div class="outline">
        <b-form @submit="submitNewServer">
          <b-form-group id="serverNameGroup" label="Server Name" label-for="serverNameInput" description="The human-readable name">
            <b-form-input id="serverNameInput" type="text" v-model="serverNewInfo.title" placeholder="New Server Name" required></b-form-input>
          </b-form-group>
          <b-form-select v-model="serverNewInfo.gameId" :options="gameOptions" class="mb-3" required />
          <b-button type="submit" variant="primary">Spin Up</b-button>
          <b-button type="reset" variant="danger">Reset</b-button>
        </b-form>
      </div>
    </b-container>
  </div>
</template>

<script>
import serverResource from '@/servers/serverResource'
export default {
  data () {
    return {
      loading: false,
      success: { show: false, message: '', maxTime: 10, curTime: 0, doGoTo: false, goToLoc: '' },
      games: [ { 'id': 1, 'name': 'ark', 'dockerImage': 'hello-world' }, { 'id': 2, 'name': 'factorio', 'dockerImage': 'hello-world' } ],
      serverNewInfo: {
        title: '',
        gameId: null
      }
    }
  },
  async created () {
    this.getGames()
  },
  methods: {
    async getGames () {
      this.loading = true
      this.games = await serverResource.getGames()
      this.loading = false
    },
    async submitNewServer (event) {
      event.preventDefault()
      this.loading = true
      try {
        await serverResource.postNewServer(this.serverNewInfo)
        this.success.show = true
        this.success.message = 'Server creation success! Redirecting you to the Dashboard.'
        this.success.curTime = this.success.maxTime
        this.success.doGoTo = true
        this.success.goToLoc = 'Dashboard'
      } catch (e) {
        console.log(e)
        alert('There was an error, check the log')
      }
      this.loading = false
    },
    countDownChanged (countDown) {
      this.success.curTime = countDown
      if (countDown === 0) {
        this.successDone()
      }
    },
    successDone () {
      if (this.success.doGoTo) {
        this.$router.push({name: this.success.goToLoc})
      }
    }
  },
  computed: {
    gameOptions: function () {
      var options = []
      options.push({ value: null, text: 'Please select an option' })
      this.games.forEach(function (e) {
        options.push({ value: e.id, text: e.name.toUpperCase() })
      })
      return options
    }
  }
}
</script>