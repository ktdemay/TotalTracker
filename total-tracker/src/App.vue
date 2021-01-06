<template>
  <div id="app">
    <b-container>
      <b-row>
        <b-col>
          <Games 
            v-if="games.length"
            :currGames="games"
            :currDate="date"
          />
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import Games from './components/Games.vue';
export default {
  name: 'app',
  components: {
    Games
  },
  data() {
    return {
      games: [],
      date: 0
    }
  },
  methods: {
    update() {
      var url = 'http://site.api.espn.com/apis/site/v2/sports/basketball/nba/scoreboard'
      // var url = 'http://localhost:8080/test.json' // TESTING

      fetch(url, {
        method: 'get'
      })
        .then((response) => {
          return response.json()
        })
        .then((jsonData) => {
          this.games = jsonData.events.slice(0);
          var date = jsonData.day.date.split('-');
          this.date = date[1] + '/' + date[2] + '/' + date[0];
        })
    }
  },
  created: function() {
    this.update();
    setInterval(this.update, 30000);
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

html {
    overflow-x: hidden;
    width: 100%;
}

body {
    overflow-x: hidden;
    width: 100%
}
</style>
